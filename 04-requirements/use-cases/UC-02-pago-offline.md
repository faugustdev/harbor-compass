# Caso de Uso: UC-02 Pago Offline (chip NFC)

## Información general

- **ID:** UC-02
- **Nombre:** Pago NFC de cliente a comercio (Void - sin internet)
- **Prioridad:** P0
- **Actores:** Cliente (App), POS Comercio (App), NFC Tag, Bóveda local
- **Pre-condición:** Cliente tiene saldo suficiente, POS configurado
- **Post-condición:** Saldo local del cliente decrementado, tx en vault local

## Diagrama de secuencia

```
Cliente App          NFC Tag            POS App            POS Vault
   │                    │                  │                    │
   │ 1. Cliente acerca  │                  │                    │
   │    tag al POS     │                  │                    │
   ├───────────────────►│                  │                    │
   │                    │ 2. Transferencia │                    │
   │                    │    NFC field on  │                    │
   │                    ├─────────────────►│                    │
   │                    │ 3. UID: 04:A3... │                    │
   │                    │                  │                    │
   │                    │                  │ 4. Lookup local   │                    │
   │                    │                  │    uid → wallet_id │
   │                    │                  │                    │
   │                    │                  │ 5. Get balance    │                    │
   │                    │                  │    from cache     │                    │
   │                    │                  │                    │
   │                    │                  │ 6. Validate       │                    │
   │                    │                  │    amount ≤ balance
   │                    │                  │                    │
   │                    │                  │ 7. Build tx        │                    │
   │                    │                  │    - tx_id (UUID)  │                    │
   │                    │                  │    - nonce         │                    │
   │                    │                  │    - amount        │                    │
   │                    │                  │    - from_uid      │                    │
   │                    │                  │    - to_merchant   │                    │
   │                    │                  │    - prev_hash     │                    │
   │                    │                  │                    │
   │                    │                  │ 8. Sign tx        │                    │
   │                    │                  │    ECDSA(private) │                    │
   │                    │                  │                    │
   │                    │                  │ 9. Update balance │                    │
   │                    │                  │    client:balance  │                    │
   │                    │                  │    - amount        │                    │
   │                    │                  │                    │
   │                    │                  │ 10. Append vault  │                    │
   │                    │                  ├───────────────────►│
   │                    │                  │ 11. Encrypt       │                    │
   │                    │                  │ 12. Store on disk │                    │
   │                    │                  │                    │
   │ 13. Mostrar OK     │                  │                    │
   │◄────────────────────┤                  │                    │
   │ "✓ Recibo exitosa"  │                  │                    │
   │                    │                  │                    │
   │ 14. Show new bal   │                  │                    │
   │                    │                  │                    │
   │                    │                  │ [POS actualiza su UI]                │
   │                    │                  │ "✓ Venta exitosa"                    │
```

## Diagrama alternativo: Pago via QR (dChip)

```
Cliente App                    POS App                   
   │                              │                     
   │                              │ 1. Genera dChip QR   │
   │                              │    (HMAC + nonce + ttl=30s)
   │                              │                     
   │ 2. Escanea QR del POS       │                     
   │    Cámara activada          │                     
   ├─────────────────────────────►│                     
   │                              │                     
   │ 3. Decodifica QR            │                     
   │    {tenant_id, device_id,    │                     
   │     nonce, ttl, hmac}       │                     
   │                              │                     
   │ 4. Verify HMAC              │                     
   │ 5. Check TTL (no expirado)  │                     
   │                              │                     
   │ 6. Si OK, build tx          │                     
   │ 7. Sign con llave privada   │                     
   │                              │                     
   │ 8. Enviar tx firmada al POS │                     
   │    (BLE / WiFi Direct / NFC) │                     
   ├──────────────────────────────►│                     
   │                              │                     
   │                              │ 10. Verify signature│                     
   │                              │ 11. Update vault     │                     
   │                              │                     
   │ 12. Confirm visual          │                     
   │◄─────────────────────────────┤                     
```

## Reglas de negocio

1. **Validación de PIN:** Requerida si `amount ≥ threshold` (configurable por tenant, default $5)
2. **Validación de saldo:** Cliente debe tener `balance >= amount + fee`
3. **Límite diario:** No exceder `wallet.daily_limit`
4. **Límite mensual:** No exceder `wallet.monthly_limit`
5. **Rate limiting:** Máx 20 tx/min por wallet
6. **Anomaly detection:** Marcar tx atípicas para revisión

## Estructura del payload de la transacción

```json
{
  "tx_id": "uuid v4",
  "type": "pay",
  "wallet_id": "uuid",
  "amount": "45.00",
  "currency": "VES",
  "counterparty_type": "merchant",
  "tenant_id": "uuid",
  "timestamp": "ISO8601",
  "nonce": "uuid v4",
  "previous_hash": "hex",
  "device_id": "uuid"
}
```

**Firma ECDSA:**

```
message = SHA-256(canonical_json(payload))
signature = ECDSA-Sign(private_key_device, message)
```

**Canonical JSON:** orden de campos fijo, sin espacios, escaped.

## Validaciones en POS antes de firmar

```kotlin
fun validateTransaction(tx: Transaction, wallet: Wallet): ValidationResult {
    // 1. Saldo suficiente
    if (wallet.balance < tx.amount + fee) {
        return InsufficientFunds(wallet.balance, tx.amount)
    }
    
    // 2. Límite diario
    val todayTotal = vault.getTodayTotal(wallet.id)
    if (todayTotal + tx.amount > wallet.dailyLimit) {
        return DailyLimitExceeded(wallet.dailyLimit, todayTotal)
    }
    
    // 3. Límite mensual
    val monthTotal = vault.getMonthTotal(wallet.id)
    if (monthTotal + tx.amount > wallet.monthlyLimit) {
        return MonthlyLimitExceeded(wallet.monthlyLimit, monthTotal)
    }
    
    // 4. Rate limiting (último minuto)
    val lastMinute = vault.getLastMinuteCount(wallet.id)
    if (lastMinute >= 20) {
        return RateLimited()
    }
    
    // 5. Wallet no congelada
    if (wallet.status != 'active') {
        return WalletBlocked()
    }
    
    return Valid()
}
```

## Manejo de errores

| Error | Acción |
|---|---|
| Tag NFC no detectado | "Acerca de nuevo" |
| UID no emparejado | "Tag no registrado" |
| Saldo insuficiente | Mostrar saldo actual |
| Límite excedido | "Excede límite" |
| PIN incorrecto | Reintentar, máx 5 |
| Tag clonado | Bloquear wallet, alerta fraude |
| Error de firma | Recargar UID lookup |

## Criterios de aceptación

- [ ] Cliente puede pagar acercando tag NFC
- [ ] Latencia < 300ms tap → confirmación
- [ ] Si monto ≥ umbral, se requiere PIN
- [ ] Tx queda en vault local encriptado
- [ ] Saldo local del cliente se decrementa
- [ ] POS comercio ve confirmación visual
- [ ] Tx persiste aunque se cierre la app

## Métricas de éxito

- 99% de pagos NFC completados exitosamente
- Latencia P95 < 300ms
- 0% de pérdida de transacciones por crash

## Sincronización posterior (referencia a UC-05)

Cuando el POS detecte internet, todas las tx pendientes se suben vía `/sync` (batch). El backend valida firmas, verifica idempotencia (por `tx_id`), actualiza balances autoritativos.