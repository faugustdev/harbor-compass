# Caso de Uso: UC-05 Sync Online (CashOut + Conciliación)

## Información general

- **ID:** UC-05
- **Nombre:** Sincronización de transacciones offline + CashOut
- **Prioridad:** P0
- **Actores:** POS App, ViCheck API, VIPPO API, Banco
- **Pre-condición:** POS tiene tx pendientes, hay conexión a internet
- **Post-condición:** Tx sincronizadas, balances actualizados, CashOut programado si aplica

## Diagrama de secuencia

```
POS App                  ViCheck API              VIPPO API               Banco
   │                          │                       │                      │
   │ 1. NetworkCallback      │                       │                      │
   │    onCapabilitiesChanged │                       │                      │
   │    (internet disponible)│                       │                      │
   │                          │                       │                      │
   │ 2. Background worker    │                       │                      │
   │    dispara sync          │                       │                      │
   │                          │                       │                      │
   │ 3. Get pending tx from  │                       │                      │
   │    vault (max 100)       │                       │                      │
   │                          │                       │                      │
   │ 4. POST /sync            │                       │                      │
   │    {batch: [...tx]}     │                       │                      │
   ├─────────────────────────►│                       │                      │
   │                          │                       │                      │
   │                          │ 5. Validate JWT       │                      │
   │                          │ 6. Validate tenant    │                      │
   │                          │ 7. For each tx:       │                      │
   │                          │    - Verify ECDSA     │                      │
   │                          │    - Check nonce      │                      │
   │                          │    - Check timestamp  │                      │
   │                          │    - Idempotencia     │                      │
   │                          │                       │                      │
   │                          │ 8. Para tx nuevas:    │                      │
   │                          │    - Append ledger    │                      │
   │                          │    - Update balance   │                      │
   │                          │                       │                      │
   │                          │ 9. Para tx duplicadas:│                      │
   │                          │    - Marcar synced    │                      │
   │                          │                       │                      │
   │ 10. Response:            │                       │                      │
   │    {batch_result: [...]} │                       │                      │
   │◄─────────────────────────┤                       │                      │
   │                          │                       │                      │
   │ 11. Update local vault   │                       │                      │
   │     (remove accepted tx)  │                       │                      │
   │                          │                       │                      │
   │ 12. Repeat for more tx   │                       │                      │
   │                          │                       │                      │
   │ [Si quedan 0 tx pendientes, considerar CashOut]   │                      │
   │                          │                       │                      │ │
   │ 13. Si balance > threshold│                       │                      │ │
   │     POST /pos/cashout    │                       │                      │ │
   ├─────────────────────────►│                       │                      │ │
   │                          │ 14. Validar saldo     │                      │ │
   │                          │ 15. Validar cuenta   │                      │ │
   │                          │                       │                      │ │
   │                          │ 16. POST /liquidacion│                      │ │
   │                          ├──────────────────────►│                      │ │
   │                          │                       │ 17. Initiate       │ │ │
   │                          │                       ├────────────────────►│ │ │
   │                          │                       │                      │ │
   │                          │                       │ 18. OK              │ │
   │                          │                       │◄─────────────────────┤ │
   │                          │                       │                      │ │
   │                          │ 19. cashout_id        │                      │ │
   │                          │◄──────────────────────┤                      │ │
   │                          │                       │                      │ │
   │ 20. Response:           │                       │                      │ │
   │     {cashout_id, status} │                       │                      │ │
   │◄─────────────────────────┤                       │                      │ │
   │                          │                       │                      │ │
   │                          │ [Background: webhook]│                      │ │
   │                          │                       │ 21. Webhook status:completed│
   │                          │◄──────────────────────┤                      │ │
   │                          │ 22. Update cashout    │                      │ │
   │                          │     status: completed │                      │ │
   │                          │ 23. Push notification │                      │ │
   │                          │                       │                      │ │
   │ 24. Push notification    │                       │                      │ │
   │◄─────────────────────────┤                       │                      │ │
   │ "Liquidación completada" │                       │                      │ │
```

## Reglas de negocio

1. **Sync idempotente:** Misma tx_id puede subirse múltiples veces sin duplicar
2. **Orden de procesamiento:** FIFO por timestamp
3. **Validación de firma:** Cada tx debe tener firma ECDSA válida
4. **Window timestamp:** tx.timestamp debe estar en últimos 5 minutos (anti-replay)
5. **Nonce único:** tx.nonce debe ser UUID v4 no usado previamente
6. **CashOut threshold:** Solo si balance >= threshold configurado por tenant

## Manejo de errores

| Error | Acción | Retry |
|---|---|---|
| Tx firma inválida | Marcar tx como invalid, alerta fraude | No |
| Tx nonce duplicado | Idempotente, marcar synced | No |
| Tx timestamp > 5min | Marcar expired, alerta | No |
| Balance insuficiente al sync | Marcar como rejected | No |
| Network error | Reintentar con backoff | Sí |
| Server 5xx | Reintentar con backoff | Sí |
| Server 4xx (validación) | Marcar tx invalid | No |

## Backoff strategy:
- Intento 1: inmediato
- Intento 2: +1 segundo
- Intento 3: +2 segundos
- Intento 4: +4 segundos
- Intento 5: +8 segundos
- Intento 6: +16 segundos
- Después: WorkManager con constraints (WiFi, charging)

## Criterios de aceptación

- [ ] POS detecta internet y dispara sync
- [ ] Sync sube tx en batch de máximo 100
- [ ] Backend acepta tx firmadas válidas
- [ ] Backend rechaza duplicados (idempotente)
- [ ] Backend rechaza firmas inválidas
- [ ] POS recibe respuesta con resultado por tx
- [ ] CashOut automático si balance >= threshold
- [ ] Tx pendientes se reintentan con WorkManager si no se completaron

## Performance

- 1000 tx pendientes sincronizadas en ≤ 30s
- 10 requests paralelos
- Latencia P95 < 1s por request

## Estados de tx después del sync

```
pending_sync → synced (si todo OK)
pending_sync → failed (si error permanente)
pending_sync → retry_later (si error temporal)
```

## Implementación backend (sync endpoint)

```python
# handlers/sync.py
async def sync_transactions(request):
    user = await authenticate(request)
    body = await request.json()
    
    results = []
    async with db.transaction():
        for tx_data in body['batch']:
            # 1. Verificar idempotencia
            existing = await tx_repo.find_by_id(tx_data['tx_id'])
            if existing:
                results.append({
                    'tx_id': tx_data['tx_id'],
                    'status': 'duplicate',
                    'message': 'already_synced'
                })
                continue
            
            # 2. Validar firma ECDSA
            is_valid = crypto.verify_signature(
                tx_data['signature'],
                canonical_payload(tx_data),
                device_public_key
            )
            if not is_valid:
                results.append({
                    'tx_id': tx_data['tx_id'],
                    'status': 'rejected',
                    'reason': 'invalid_signature'
                })
                continue
            
            # 3. Validar timestamp window
            tx_time = parse_iso(tx_data['timestamp'])
            if abs((now() - tx_time).total_seconds()) > 300:
                results.append({
                    'tx_id': tx_data['tx_id'],
                    'status': 'rejected',
                    'reason': 'expired_timestamp'
                })
                continue
            
            # 4. Validar saldo
            wallet = await wallet_repo.get(user.wallet_id)
            if wallet.balance < Decimal(tx_data['amount']):
                results.append({
                    'tx_id': tx_data['tx_id'],
                    'status': 'rejected',
                    'reason': 'insufficient_funds'
                })
                continue
            
            # 5. Append transacción
            new_tx = await tx_repo.create({
                'id': tx_data['tx_id'],
                'wallet_id': user.wallet_id,
                'tenant_id': user.tenant_id,
                'type': tx_data['type'],
                'amount': tx_data['amount'],
                'currency': tx_data['currency'],
                'status': 'synced',
                'offline_signature': tx_data['signature'],
                'offline_nonce': tx_data['nonce'],
                'offline_timestamp': tx_data['timestamp'],
                'synced_at': now(),
                'device_id': body['device_id']
            })
            
            # 6. Update balance
            new_balance = await ledger_service.append_tx(new_tx)
            
            results.append({
                'tx_id': tx_data['tx_id'],
                'status': 'accepted',
                'new_balance': str(new_balance)
            })
    
    return response.json({
        'batch_result': results,
        'new_pending_count': await tx_repo.count_pending(user.wallet_id)
    })
```

## Métricas de éxito

- 100% de tx válidas se sincronizan en primer intento
- 0% de duplicación al subir 2+ veces la misma tx
- 95% de sync batch completados en < 30s
- 0% de pérdida de tx por crashes