# Caso de Uso: UC-01 CashIn Online

## Información general

- **ID:** UC-01
- **Nombre:** CashIn - Carga de saldo desde banco del cliente
- **Prioridad:** P0
- **Actores:** Cliente (App), ViCheck API, VIPPO API, Banco
- **Pre-condición:** Usuario autenticado en ViCheck, con cuenta bancaria VIPPO asociada
- **Post-condición:** Saldo del usuario incrementado, transacción `cashin` en estado `synced`

## Diagrama de secuencia

```
Cliente                ViCheck API              VIPPO API               Banco
   │                       │                       │                      │
   │ 1. POST /cashin        │                       │                      │
   │ {amount: 200, ...}     │                       │                      │
   ├───────────────────────►│                       │                      │
   │                       │ 2. Validar user        │                      │
   │                       │    Verificar límite    │                      │
   │                       │    Generar tx_id       │                      │
   │                       │                       │                      │
   │                       │ 3. POST /validacion   │                      │
   │                       │    de pago             │                      │
   │                       ├──────────────────────►│                      │
   │                       │                       │ 4. Validar cuenta    │
   │                       │                       ├─────────────────────►│
   │                       │                       │                      │
   │                       │                       │ 5. OK                │
   │                       │                       │◄─────────────────────┤
   │                       │                       │                      │
   │                       │ 6. cashin_id           │                      │
   │                       │◄──────────────────────┤                      │
   │                       │    status: pending     │                      │
   │ 7. 202 Response        │                       │                      │
   │◄───────────────────────┤                       │                      │
   │ {cashin_id, pending}  │                       │                      │
   │                       │                       │                      │
   │ [Background: poll cada 5s]                       │                      │
   │                       │                       │ 8. Initiate transfer │
   │                       │                       ├─────────────────────►│
   │                       │                       │                      │
   │                       │                       │ 9. Transfer OK       │
   │                       │                       │◄─────────────────────┤
   │                       │                       │                      │
   │                       │                       │ 10. Webhook          │
   │                       │◄──────────────────────┤                      │
   │                       │     {status:completed} │                      │
   │                       │                       │                      │
   │                       │ 11. Update balance    │                      │
   │                       │ 12. Append ledger     │                      │
   │                       │ 13. Push notif       │                      │
   │                       │                       │                      │
   │ 14. Push notification │                       │                      │
   │◄───────────────────────┤                       │                      │
   │ "Saldo acreditado"   │                       │                      │
   │                       │                       │                      │
   │ 15. GET /balance (opcional)                    │                      │
   ├───────────────────────►│                       │                      │
   │                       │                       │                      │
   │ 16. {balance: 12500}  │                       │                      │
   │◄───────────────────────┤                       │                      │
```

## Flujo alternativo

### A1: CashIn fallido (banco rechaza)

```
Banco
   │
   │ 9. Transfer rejected (saldo insuficiente)
   ├────────────────────────────────────► VIPPO
   │
VIPPO
   │ 10. Webhook {status: failed, reason: "insufficient_funds"}
   ├────────────────────────────────────► ViCheck
   │
ViCheck
   │ 11. Marcar tx como `failed`
   │ 12. Push notif al cliente
   │
Cliente
   │ 14. "No pudimos procesar la recarga. Verifica tu saldo bancario."
```

### A2: Webhook no llega (timeout 60s)

```
Cliente
   │   │ Cliente: GET /cashin/{id} cada 10s
   │   │
ViCheck
   │   │ Backend: query vippo API para verificar estado
   │   │
   │   │ Si timeout > 60s: marcar como `unknown`, requiere
   │   │ intervención manual de soporte
```

## Reglas de negocio

1. **Monto mínimo:** 100 VES (configurable por tenant)
2. **Monto máximo CashIn sin KYC completo:** 500 USD/mes (regulatorio)
3. **CashIn máximo por día:** configurable por tenant (default 5000 VES)
4. **Comisión:** ninguna para el cliente (ya se cobra al tenant)
5. **Timeout:** 60 segundos esperando webhook, después polling manual

## Campos del request

```json
{
  "amount": "string (decimal)",
  "currency": "VES",
  "source_account_id": "uuid",
  "source_bank_code": "string (4 digits)"
}
```

## Campos de respuesta exitosa

```json
{
  "cashin_id": "uuid",
  "status": "pending | completed | failed",
  "amount": "200.00",
  "currency": "VES",
  "new_balance": "12500.00",
  "vippo_cashin_id": "string",
  "completed_at": "ISO8601"
}
```

## Manejo de errores

| Código | Significado | Acción cliente |
|---|---|---|
| 400 | amount inválido | Mostrar error validación |
| 401 | Token expirado | Refresh + reintentar |
| 409 | CashIn duplicado | Idempotencia |
| 422 | Monto excede límite | Mostrar "excede límite diario" |
| 429 | Rate limit | Mostrar "espera N minutos" |
| 503 | VIPPO API down | Mostrar "intenta más tarde" |

## Implementación backend

```python
# handlers/cashin.py
async def create_cashin(request):
    user = await authenticate(request)
    body = await request.json()
    
    # Validar monto
    if Decimal(body['amount']) < MIN_CASHIN:
        raise HTTPBadRequest("Montoo debajo del MIN_CASHIN")
    
    # Validar límite diario
    today_total = await get_today_cashin_total(user.wallet_id)
    if today_total + Decimal(body['amount']) > user.daily_limit:
        raise HTTPUnprocessableEntity("Excede límite")
    
    # Llamar VIPPO
    vippo_resp = await vippo_client.init_cashin(
        user_id=user.vippo_user_id,
        amount=body['amount'],
        source_account=body['source_account_id']
    )
    
    # Crear tx local
    tx = await transaction_repo.create(
        id=uuid.uuid4(),
        wallet_id=user.wallet_id,
        type='cashin',
        amount=body['amount'],
        status='pending',
        vippo_cashin_id=vippo_resp.id
    )
    
    return response.json({
        'cashin_id': str(tx.id),
        'status': 'pending'
    }, status=202)
```

## Criterios de aceptación

- [ ] El usuario puede iniciar CashIn con monto ≥ 100 VES
- [ ] Backend rechaza montos que excedan límite diario
- [ ] El cliente recibe confirmación push al completarse
- [ ] Si webhook no llega en 60s, polling manual retorna estado
- [ ] CashIn fallido NO acredita saldo
- [ ] Tx queda en `audit_log`
- [ ] Dashboard comercio se actualiza al instante (en su tenant)

## Métricas de éxito

- 95% de CashIn completados en < 5 segundos
- < 1% de CashIn fallidos por error del sistema
- 100% de CashIn quedan en `audit_log`