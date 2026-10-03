# ViCheck MVP — Arquitectura Técnica (Modelo corregido)

## 1. Visión general

ViCheck es una **capa de orquestación offline-first** para pagos entre Cliente → Aliado. ViCheck **NO custodia dinero** — solo registra, valida y sincroniza.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            CLIENTE (App Android)                            │
│  Wallet lógica (NO contiene dinero real, solo "autorización")                │
│                                                                             │
│  ┌──────────────────┐    ┌────────────────────────────────────┐            │
│  │  NFC Module      │    │          Core App (Kotlin)         │            │
│  │  - Read UID      │    │  - CashIn UI                       │            │
│  │  - Foreground    │    │  - Payment UI                      │            │
│  │    Dispatch      │    │  - QR Generator (dChip)            │            │
│  └────────┬─────────┘    │  - State machine (online/offline)  │            │
│           │              └─────────────────┬───────────────────┘            │
│           │                                  │                              │
│  ┌────────▼──────────────────────────────────▼───────────────────┐         │
│  │                   Offline-First Engine                          │       │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐│       │
│  │  │ Local Ledger │  │ Tx Queue    │  │ Encrypted Vault (AES-256) ││       │
│  │  │ (crédito)    │  │ (pending)   │  │ - pending_transactions   ││       │
│  │  └─────────────┘  └─────────────┘  │ - keys, signatures       ││       │
│  │                                      └─────────────────────────┘│       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
└──────────────────────────────────┬───────────────────────────────────────────┘
                                   │ P2P / NFC tap / QR scan
┌──────────────────────────────────▼──────────────────────────────────────────┐
│                          POS ALIADO (App Android)                            │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │                    POS Logic (Kotlin x Compose)                  │        │
│  │  - read NFC UID from client                                      │        │
│  │  - decode dChip QR from client                                   │        │
│  │  - sign transaction (ECDSA P-256)                               │        │
│  │  - update local vault                                            │        │
│  │  - (NO descuenta saldo: solo registra la venta)                 │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                                                                              │
└──────────────────────────────────┬───────────────────────────────────────────┘
                                   │ Internet when online
┌──────────────────────────────────▼───────────────────────────────────────────┐
│                          BACKEND ViCheck (AWS SAM)                            │
│                                                                              │
│  ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐  │
│  │  API Gateway        │  │  Lambda Functions  │  │  EventBridge API     │  │
│  │  (REST + WS)        │  │  (Python 3.14)     │  │  (async events)     │  │
│  └─────────┬────────────┘  └─────────┬──────────┘  └─────────┬──────────┘  │
│            │                          │                       │                      │
│  ┌─────────▼──────────────────────────▼───────────────────────▼──────────┐│
│  │                          Core Services                                  ││
│  │  - CashInService         - ReconciliationService    - SyncService       ││
│  │  - TenantService        - BankIntegrationService   - NotificationSvc   ││
│  └─────────┬──────────────────────────────────────────────┬──────────────┘│
│            │                              │                              │        │
│  ┌─────────▼───────────────┐  ┌────────▼────────────────┐                │        │
│  │  PostgreSQL (RDS)        │  │  Redis (cache)            │                │        │
│  │  - tenants               │  │  - hot balance mirror   │                │        │
│  │  - users                 │  │  - rate limit            │                │        │
│  │  - wallets (lógicas)     │  │  - session cache         │                │        │
│  │  - transactions          │  └──────────────────────────┘                │        │
│  │  - sync_log (append-only)│                                                  │        │
│  └──────────────────────────┘                                                  │        │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │              External: VIPPO APIs                               │        │
│  │  - /validacion de pago (validación contra banco del cliente)   │        │
│  │  - /cuentas (gestión de cuentas bancarias)                    │        │
│  │  - /transferencia (transferencia entre aliados)                │        │
│  └─────────────────────────────────────────────────────────────────┘        │
└──────────────────────────────────────────────────────────────────────────────┘
```

## 2. Las 3 fases del OffVi Sync Engine

### Fase 1: CashIn (Online)

El cliente carga saldo en su wallet ViCheck. Esta fase **requiere internet**. El dinero va directo del banco del cliente a la cuenta del Aliado. ViCheck solo registra.

```mermaid
sequenceDiagram
    participant U as Cliente (App)
    participant V as ViCheck API
    participant P as VIPPO API
    participant BC as Banco del Cliente
    participant BA as Banco del Aliado

    U->>V: POST /cashin {amount: 200, aliado_id: 'x'}
    V->>P: POST /validacion de pago {user, amount}
    P->>BC: validar saldo
    BC-->>P: 200 OK
    P-->>V: {validacion_id, status: 'pending'}
    V-->>U: 202 Accepted {cashin_id}

    Note over P,BA: VIPPO ejecuta la transferencia
    BC->>BA: transferir(amount)
    BA-->>BC: transfer OK
    P-->>V: webhook {status: 'completed'}

    V->>V: registra en wallet lógica
    V->>V: crea transacción tipo 'cashin'
    V-->>U: push notification 'saldo disponible'
```

**Eventualidades:**
- Si el webhook no llega en 30s, el cliente puede pull con `GET /cashin/:id`
- Si falla la carga, no se acredita saldo en ViCheck
- Auditoría: cada carga queda registrada en `transactions` con `source = 'vippo_cashin'`

### Fase 2: Transacción offline (Void)

El cliente paga al Aliado sin internet. Hay dos medios físicos:

#### 2a. NFC (chip-to-phone)

```mermaid
sequenceDiagram
    participant C as Client App
    participant NFC as NFC Tag
    participant P as POS Aliado
    participant V as Local Vault

    C->>NFC: tap (cliente acerca pulsera/tarjeta/celular)
    NFC-->>C: UID: 04:A3:B2:C1:D4:E5:F6
    C->>C: search local wallet by UID
    C->>C: valida saldo local (cache)
    C->>C: sign tx {from_uid, to_aliado, amount, nonce, ts}
    C->>P: tx pushed via NFC (NDEF) o BLE
    P->>P: verify signature
    P->>P: registra venta localmente (NO descuenta saldo)
    P->>V: encrypt and store
    P-->>C: confirmation via NFC/NDEF
```

#### 2b. dChip QR (phone-to-phone)

```mermaid
sequenceDiagram
    participant C as Client App
    participant P as POS Aliado

    P->>P: generates dChip QR (HMAC + nonce + ttl)
    Note over P: QR se renueva cada 30s
    C->>P: scan QR
    C->>C: verify HMAC + ttl
    C->>P: POST signed tx via local WiFi/BLE o QR secondary channel
    P->>P: verify + record
```

**El PIN de confirmación:** Cada transacción offline con monto significativo (≥ configurable por tenant) requiere **PIN entry en el celular del cliente** antes de firmar.

**Estado local:**
- `Local Ledger (SQLite + SQLCipher)`: wallet lógica cacheada
- `Pending Transactions Queue`: cola de TX firmadas esperando sync
- `Key Store`: llaves ECDSA del dispositivo

### Fase 3: Sync + Conciliación (Online)

Cuando el dispositivo detecta red (NetworkCallback en Android), ejecuta el sync. NO hay cashout del Aliado — el dinero ya está en su banco.

```mermaid
sequenceDiagram
    participant D as Cliente (o POS)
    participant V as ViCheck API
    participant DB as DB
    participant P as VIPPO API

    D->>V: POST /sync {batch: [...pending_txs]}
    V->>V: validate batch (firmas, secuencia, no duplicados)
    loop for each tx
        V->>DB: registra transacción (sync_log append-only)
        V->>DB: actualiza wallet lógica del cliente
    end
    V-->>D: 200 OK {batch_result, new_wallet_state}

    Note over V,P: Si hay transferencia entre aliados
    V->>P: POST /transferencia {aliado_a, aliado_b, amount}
    P-->>V: {transfer_id, status}
```

## 3. Componentes detallados

### 3.1 Offline-First Engine (Android)

```kotlin
// Estructura de packages
com.vippo.vicheck/
├── core/
│   ├── nfc/                  ← lectura/escritura NFC
│   ├── qr/                   ← generador dChip + scanner
│   ├── crypto/               ← ECDSA, ECDH, AES-256
│   ├── storage/              ← Room + SQLCipher
│   └── sync/                 ← NetworkCallback, retry, backoff
├── data/
│   ├── model/                ← Wallet (lógica), Transaction, User, Tenant
│   └── repository/           ← Local + Remote data sources
├── domain/
│   ├── usecase/              ← CashIn, Pay, Sync, TransferBetweenAliados
│   └── state/                ← State machine Online/Offline/Partial
├── feature/
│   ├── onboarding/
│   ├── wallet/                ← balance (lógico), history
│   ├── pay/                  ← pagar a aliado
│   ├── topup/                ← cashin flow
│   ├── transfer/              ← transferir entre aliados
│   └── settings/
└── di/                       ← Hilt modules
```

### 3.2 Backend (AWS SAM + Lambda Python 3.14)

```
vicheck-api/
├── template.yaml             ← SAM template
├── src/
│   ├── handlers/
│   │   ├── cashin.py         ← POST /cashin
│   │   ├── sync.py           ← POST /sync
│   │   ├── transfer.py       ← POST /transfer (entre aliados)
│   │   ├── balance.py        ← GET /balance/:uid
│   │   ├── webhook_vippo.py  ← inbound webhook
│   │   └── reconciliation.py ← CRON
│   ├── services/
│   │   ├── wallet.py         ← wallet lógica (NO dinero real)
│   │   ├── tenant.py
│   │   ├── vippo_client.py   ← HTTP client a VIPPO APIs
│   │   └── reconciliation.py
│   ├── models/
│   │   └── pydantic_models.py
│   ├── db/
│   │   ├── migrations/       ← Alembic
│   │   └── connection.py     ← psycopg2 pool
│   └── utils/
│       └── auth.py            ← JWT validation
├── tests/
│   └── unit/
└── requirements.txt
```

### 3.3 Base de datos (PostgreSQL)

```sql
CREATE TABLE tenants (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  vippo_merchant_id TEXT UNIQUE,
  bank_account_id TEXT,
  status TEXT DEFAULT 'active',
  branding JSONB,
  config JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE users (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  phone TEXT UNIQUE NOT NULL,
  email TEXT,
  full_name TEXT,
  cedula TEXT,
  kyc_status TEXT DEFAULT 'pending',
  vippo_user_id TEXT UNIQUE,
  pin_hash TEXT,
  biometric_enabled BOOLEAN DEFAULT false,
  status TEXT DEFAULT 'active',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE wallets (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  balance DECIMAL(18,4) NOT NULL DEFAULT 0,  -- saldo lógico (crédito contra aliado)
  currency TEXT NOT NULL DEFAULT 'VES',
  status TEXT DEFAULT 'active',
  daily_limit DECIMAL(18,4) DEFAULT 1000.00,
  monthly_limit DECIMAL(18,4) DEFAULT 5000.00,
  offline_pending DECIMAL(18,4) DEFAULT 0,  -- saldo reservado en offline
  offline_max_amount DECIMAL(18,4) DEFAULT 500.00,  -- tope configurado (sincronizado backend)
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, tenant_id, currency)
);

CREATE TABLE transactions (
  id UUID PRIMARY KEY,
  wallet_id UUID NOT NULL REFERENCES wallets(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  type TEXT NOT NULL,  -- 'cashin', 'pay', 'transfer', 'fee'
  amount DECIMAL(18,4) NOT NULL,
  currency TEXT NOT NULL,
  counterparty_aliado_id UUID REFERENCES tenants(id),
  status TEXT NOT NULL DEFAULT 'pending_sync',
  offline_signature TEXT,
  offline_nonce UUID,
  offline_timestamp TIMESTAMPTZ,
  synced_at TIMESTAMPTZ,
  vippo_cashin_id TEXT,
  vippo_transfer_id TEXT,
  device_id UUID REFERENCES devices(id),
  metadata JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  CONSTRAINT chk_type CHECK (type IN ('cashin', 'pay', 'transfer', 'fee')),
  CONSTRAINT chk_status CHECK (status IN ('pending_sync', 'synced', 'reconciled', 'reversed', 'failed'))
);

CREATE TABLE sync_log (
  id BIGSERIAL PRIMARY KEY,
  device_id UUID NOT NULL,
  user_id UUID NOT NULL,
  batch_id UUID NOT NULL,
  tx_count INTEGER NOT NULL,
  first_tx_id UUID,
  last_tx_id UUID,
  sync_started_at TIMESTAMPTZ,
  sync_completed_at TIMESTAMPTZ,
  status TEXT NOT NULL,
  error_detail TEXT
);

CREATE TABLE transfers (
  id UUID PRIMARY KEY,
  from_aliado_id UUID NOT NULL REFERENCES tenants(id),
  to_aliado_id UUID NOT NULL REFERENCES tenants(id),
  user_id UUID NOT NULL REFERENCES users(id),
  amount DECIMAL(18,4) NOT NULL,
  status TEXT NOT NULL,  -- 'pending', 'completed', 'failed'
  vippo_transfer_id TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  completed_at TIMESTAMPTZ,
  metadata JSONB
);

CREATE INDEX idx_tx_wallet_pending ON transactions(wallet_id) WHERE status = 'pending_sync';
CREATE INDEX idx_tx_tenant_created ON transactions(tenant_id, created_at DESC);
CREATE INDEX idx_tx_offline_nonce ON transactions(offline_nonce) WHERE offline_nonce IS NOT NULL;
CREATE INDEX idx_transfers_user ON transfers(user_id, created_at DESC);
```

### 3.4 Criptografía

| Uso | Algoritmo | Implementación |
|---|---|---|
| Firma tx offline | ECDSA P-256 | Bouncy Castle (Android) / python-ecdsa (backend) |
| Encriptación vault local | AES-256-GCM | Tink (Android) / cryptography (Python) |
| Hash dChip | HMAC-SHA256 | libsodium / Tink |
| Llave dispositivo↔backend | ECDH P-256 | secp256r1 |
| Storage password | Argon2id | argon2-cffi (Python) |

**Generación de llaves:** El backend genera el par de llaves ECDSA en el registro del dispositivo. La llave privada se entrega al dispositivo sobre un canal seguro (autenticación mutua con TLS) y se almacena en `EncryptedSharedPreferences` de Android.

## 4. Flujo de datos completo (pago offline + sync)

```
CLIENTE                              POS ALIADO                              BACKEND
   │                                      │                                    │
   │ 1. Tap pulsera al POS                │                                    │
   │─────────────────────────────────────►│                                    │
   │                                      │                                    │
   │   2. POS lee UID                     │                                    │
   │   3. POS consulta balance local     │                                    │
   │      (wallet lógica cacheada)        │                                    │
   │                                      │                                    │
   │   4. POS muestra monto + aliado       │                                    │
   │◄─────────────────────────────────────┤                                    │
   │                                      │                                    │
   │   5. Cliente ingresa PIN             │                                    │
   │   6. Cliente firma tx                │                                    │
   │─────────────────────────────────────►│                                    │
   │                                      │                                    │
   │   7. POS verifica firma              │                                    │
   │   8. POS REGISTRA la venta           │                                    │
   │      (NO descuenta saldo aquí)       │                                    │
   │   9. POS almacena en vault local     │                                    │
   │                                      │                                    │
   │   10. POS muestra OK al comercio +   │                                    │
   │       cliente                        │                                    │
   │◄─────────────────────────────────────┤                                    │
   │                                      │                                    │
   │                            [NO INTERNET]    [INTERNET VUELVE]            │
   │                                      │                                    │
   │                                      │  11. POST /sync {batch: [...tx]}  │
   │                                      ├──────────────────────────────────► │
   │                                      │                                    │
   │                                      │  12. Backend valida batch         │
   │                                      │  13. Backend registra tx          │
   │                                      │  14. Backend actualiza wallet      │
   │                                      │      lógica del cliente            │
   │                                      │                                    │
   │                                      │  15. 200 OK {new_wallet_state}    │
   │                                      │◄──────────────────────────────────┤
   │                                      │                                    │
   │                                      │  [Si hay transferencia entre     │
   │                                      │   aliados, se ejecuta vía VIPPO]  │
```

## 5. Seguridad

| Capa | Medida |
|---|---|
| **Transporte** | TLS 1.3 en todas las llamadas HTTP/HTTPS |
| **Autenticación** | JWT firmado por backend, refresh tokens con rotación |
| **Autorización** | RBAC por tenant: cliente solo ve su wallet lógica, aliado ve sus ventas |
| **Vault local** | AES-256-GCM, llave derivada de master + device-bound key |
| **Firmas tx** | Cada tx tiene nonce + firma ECDSA, previene duplicación |
| **Anti-replay** | Nonce + timestamp en cada tx, ventana de aceptación configurable |
| **Device binding** | Cada dispositivo tiene par de llaves único registrado en backend |
| **Audit log** | sync_log append-only, todas las operaciones registradas |
| **Rate limiting** | Por usuario, por tenant, configurable |
| **Multi-tenancy** | `tenant_id` en todas las queries, nunca cruzado |
| **Pre-desinstalación** | App fuerza sync + muestra alerta explícita "perderás X Bs" |

## 6. Performance y capacidad esperada

| Métrica | Target MVP | Diseño |
|---|---|---|
| Latencia NFC tap → confirmación | < 300 ms | NFC read + local lookup + firma |
| Sync time (1000 tx pendientes) | < 30 s | Batch upload + idempotente design |
| TPS por nodo POS | ≥ 5 TPS | SQLite local, batch writes |
| Backend TPS | ≥ 100 TPS | Lambda + Postgres + connection cache |
| Cold start Android app | < 2 s | Compose + Hilt + lazy init |
| Tamaño APK | < 50 MB | Sin recursos pesados |

## 7. Despliegue

### 7.1 Android
- Google Play Internal Testing track (closed beta)
- Fastlane para build + deploy automático
- Crashlytics + Firebase Analytics
- Play Console: requerirá privacy policy + data safety form

### 7.2 Backend
- AWS SAM CLI: `sam build && sam deploy`
- Multi-environment: `dev`, `staging`, `prod`
- RDS PostgreSQL + ElastiCache Redis
- API Gateway + Lambda Python 3.14
- CloudWatch + X-Ray para observabilidad
- Secrets Manager para llaves

### 7.3 CI/CD
- GitHub Actions
- Tests: pytest (backend), JUnit + Espresso (Android)
- Lint: ruff (Python), ktlint + detekt (Kotlin)
- Deploy: automático a staging, manual approval a prod

---

**Próximo documento:** `03-tech-stack.md` — stack propuesto con justificación por elección.