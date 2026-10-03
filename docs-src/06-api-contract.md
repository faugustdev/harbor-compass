# ViCheck MVP — Contrato de API

## 1. Convenciones

- **Base URL:** `https://api.vicheck.vippo.io/v1`
- **Auth:** Bearer JWT en header `Authorization`
- **Content-Type:** `application/json`
- **Versionado:** URL `/v1/`, breaking changes → `/v2/`
- **Errores:** RFC 7807 (Problem Details for HTTP APIs)

## 2. Endpoints públicos

## 2.1 Auth

#### `POST /auth/request-otp`
#### `POST /auth/verify-otp`
#### `POST /auth/refresh`

### 2.2 CashIn (online)

#### `POST /cashin`
Inicia CashIn. Valida contra banco del cliente, transfiere dinero a cuenta del Aliado, acredita wallet lógica del cliente.

**Request:**
```json
{
  "amount": "200.00",
  "currency": "VES",
  "source_bank_account_id": "uuid",
  "source_bank_code": "0102",
  "aliado_id": "uuid"
}
```

### 2.3 Sync (pagos offline)

#### `POST /sync`
Sincroniza transacciones offline pendientes. NO descuenta saldo aquí — ya se descontó en el dispositivo. Aquí solo registra y concilia.

**Request:**
```json
{
  "device_id": "uuid",
  "batch": [
    {
      "tx_id": "uuid",
      "type": "pay",
      "amount": "45.00",
      "currency": "VES",
      "counterparty_aliado_id": "uuid-aliado",
      "timestamp": "2026-10-02T14:00:00Z",
      "nonce": "uuid",
      "signature": "0x9f4a..."
    }
  ]
}
```

**Response 200:**
```json
{
  "batch_result": [
    {
      "tx_id": "uuid",
      "status": "accepted",
      "synced_at": "2026-10-02T14:35:00Z",
      "new_balance": "12455.00"
    }
  ],
  "new_pending_count": 0
}
```

**Response items por tx:**
- `accepted`: tx sincronizada, balance actualizado
- `duplicate`: tx ya existía (idempotencia)
- `rejected`: tx rechazada (firma inválida, saldo insuficiente, etc.)

#### `GET /balance`
Balance actual del usuario autenticado.

**Response 200:**
```json
{
  "wallet_id": "uuid",
  "balance": "12455.00",
  "currency": "VES",
  "pending_sync": 2,
  "as_of": "2026-10-02T14:35:00Z"
}
```

#### `GET /transactions`
Historial de transacciones.

**Query params:** `from`, `to`, `type`, `limit` (max 100), `offset`

**Response 200:**
```json
{
  "items": [
    {
      "tx_id": "uuid",
      "type": "pay",
      "amount": "-45.00",
      "currency": "VES",
      "counterparty": "Terminal La Bandera",
      "status": "synced",
      "timestamp": "2026-10-02T14:00:00Z"
    }
  ],
  "next_offset": null,
  "total": 87
}
```

### 2.4 NFC Tags

#### `POST /nfc-tags/pair`
Empareja un tag NFC con la wallet del usuario.

**Request:**
```json
{
  "uid": "04:A3:B2:C1:D4:E5:F6",
  "name": "Mi pulsera",
  "pin": "123456"
}
```

**Response 200:**
```json
{
  "id_tag": "uuid",
  "uid": "04:A3:B2:C1:D4:E5:F6",
  "name": "Mi pulsera",
  "paired_at": "2026-10-02T14:35:00Z"
}
```

#### `GET /nfc-tags`
Lista tags emparejados del usuario.

#### `DELETE /nfc-tags/{id_tag}`
Desempareja un tag.

### 2.5 Webhooks (inbound, desde VIPPO)

#### `POST /webhooks/vippo/cashin`
Notificación de estado CashIn.

```json
{
  "vippo_cashin_id": "uuid",
  "status": "completed" | "failed",
  "amount": "200.00",
  "failure_reason": null
}
```

## 3. Endpoints POS (merchant)

### 3.1 POS Auth

#### `POST /pos/auth/login`
POS login (separado del cliente).

### 3.2 POS Cobro

#### `POST /pos/transactions`
POS reporta una transacción que ya fue procesada offline.

#### `GET /pos/transactions`
POS consulta historial de ventas.

#### `GET /pos/dashboard`
Métricas del POS: ventas hoy, semana, mes.

### 3.3 POS Reporte (NO hay CashOut)

**NO existe `/pos/cashout`** — el dinero del Aliado ya está en su banco desde el CashIn. El Aliado solo consulta reportes de ventas.

## 4. Endpoints Admin (interno)

#### `GET /admin/tenants`
Lista de tenants.

#### `POST /admin/tenants`
Crea tenant.

#### `GET /admin/transactions/{tx_id}`
Detalle cross-tenant de una transacción.

#### `POST /admin/wallets/{id}/freeze`
Congela una wallet.

#### `POST /admin/transactions/{id}/reverse`
Reversa una transacción.

## 5. Códigos de error

| Código | Significado |
|---|---|
| 400 | Bad Request (request inválido) |
| 401 | Unauthorized (token expirado/inválido) |
| 403 | Forbidden (permisos insuficientes) |
| 404 | Not Found |
| 409 | Conflict (e.g., tx duplicada) |
| 422 | Unprocessable Entity (validación falló) |
| 429 | Too Many Requests (rate limit) |
| 500 | Internal Server Error |
| 503 | Service Unavailable (mantenimiento) |

**Formato de error (RFC 7807):**
```json
{
  "type": "https://vicheck.vippo.io/errors/insufficient-funds",
  "title": "Insufficient funds",
  "status": 422,
  "detail": "Your wallet balance is less than the transaction amount",
  "instance": "/v1/transactions",
  "current_balance": "10.00",
  "required_amount": "45.00"
}
```

## 6. Rate limits

| Endpoint | Límite |
|---|---|
| `/auth/*` | 10 req/min por IP |
| `/sync` | 60 req/min por wallet |
| `/cashin` | 20 req/hora por wallet |
| `/balance` | 120 req/min por wallet |
| Otros | 300 req/min por usuario |

## 7. Versionado

- URL: `/v1/`
- Breaking changes → `/v2/`
- Mantener `/v1/` activo 12 meses después de `/v2/`

---

**Próximo documento:** `07-time-estimate.md`