# ViCheck MVP — Modelo de Datos (PostgreSQL)

## Diagrama ER simplificado

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│   tenants    │       │    users     │       │   wallets    │
├──────────────┤       ├──────────────┤       ├──────────────┤
│ id (PK)      │◄──┐   │ id (PK)      │◄──┐   │ id (PK)      │
│ name         │   │   │            │   │   │ user_id (FK) │─┘
│ vippo_merchant_id ││  tenant_id (FK)│   │   │ tenant_id (FK)│
│ bank_account_id  ││  phone (UNIQUE) │   │   │ balance      │
│ status        ││   │ email          │   │   │ currency     │
│ created_at    ││   │ full_name      │   │   │ status       │
└──────────────┘│  │ kyc_status       │   │   │ created_at   │
                │  │ vippo_user_id    │   │   └──────┬───────┘
                │  │ created_at       │   │          │
                │  └──────────────┘   │   │          │
                │                     │   │          │
                │  ┌──────────────────┘   │          ▼
                │  │                      │   ┌──────────────┐
                │  │                      │   │ transactions │
                │  │                      │   ├──────────────┤
                │  │                      │   │ id (PK)      │
                │  │                      │   │ wallet_id(FK)│
                │  │                      │   │ tenant_id(FK)│
                │  │                      │   │ type         │
                │  │                      │   │ amount       │
                │  │                      │   │ currency     │
                │  │                      │   │ counterparty │
                │  │                      │   │ status       │
                │  │                      │   │ offline_sig  │
                │  │                      │   │ nonce        │
                │  │                      │   │ created_at   │
                │  │                      │   │ synced_at    │
                │  │                      │   │ vippo_cashin_id│
                │  │                      │   │ vippo_cashout_id│
                │  │                      │   └──────┬───────┘
                │  │                      │          │
                │  │                      │          ▼
                │  │                      │   ┌──────────────┐
                │  │                      │   │   ledger     │
                │  │                      │   ├──────────────┤
                │  │                      │   │ id (PK)      │
                │  │                      │   │ tx_id (FK)   │
                │  │                      │   │ wallet_id    │
                │  │                      │   │ direction    │
                │  │                      │   │ amount       │
                │  │                      │   │ balance_after│
                │  │                      │   │ recorded_at  │
                │  │                      │   └──────────────┘
                │  │                      │
                ▼  └──────────────────────┘
                
┌──────────────┐       ┌──────────────┐
│  devices     │       │  audit_log   │
├──────────────┤       ├──────────────┤
│ id (PK)      │       │ id (PK)      │
│ user_id (FK) │       │ tenant_id    │
│ tenant_id    │       │ actor_id     │
│ device_type  │       │ action       │
│ public_key   │       │ resource     │
│ device_name  │       │ details (JSON)│
│ status       │       │ created_at   │
│ last_seen_at │       └──────────────┘
│ created_at   │
└──────────────┘

┌──────────────┐       ┌──────────────┐
│   cashouts   │       │  anomalies   │
├──────────────┤       ├──────────────┤
│ id (PK)      │       │ id (PK)      │
│ tenant_id    │       │ wallet_id    │
│ amount       │       │ rule         │
│ status       │       │ severity     │
│ vippo_cashout│       │ description  │
│ requested_at │       │ created_at   │
│ completed_at │       │ resolved_at  │
│ destination  │       └──────────────┘
└──────────────┘
```

## Tablas detalladas

### tenants
Representa cada comercio/prestador afiliado a ViCheck.

```sql
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,                    -- URL-friendly identifier
  vippo_merchant_id TEXT UNIQUE,                -- FK al sistema VIPPO
  vippo_bank_account_id TEXT,                   -- cuenta destino CashOut
  fee_percentage DECIMAL(5,4) DEFAULT 0.0200,   -- fee base 2%
  pin_required_threshold DECIMAL(18,4) DEFAULT 5.00, -- USD threshold
  status TEXT NOT NULL DEFAULT 'active',        -- active, suspended, closed
  branding JSONB,                    -- { logo_url, primary_color, ... }
  config JSONB,                      -- { auto_cashout_threshold, ... }
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_tenants_status ON tenants(status) WHERE status = 'active';
CREATE INDEX idx_tenants_vippo ON tenants(vippo_merchant_id);
```

### users
Usuarios finales (clientes) que usan ViCheck como billetera.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  phone TEXT UNIQUE NOT NULL,         -- +58412XXXXXXX
  email TEXT,
  full_name TEXT NOT NULL,
  cedula TEXT,                       -- V-12345678
  kyc_status TEXT NOT NULL DEFAULT 'pending',  -- pending, basic, full
  vippo_user_id TEXT UNIQUE,         -- FK al user de VIPPO
  pin_hash TEXT,                     -- bcrypt del PIN
  biometric_enabled BOOLEAN DEFAULT false,
  status TEXT NOT NULL DEFAULT 'active',  -- active, blocked, deleted
  language TEXT DEFAULT 'es',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_tenant ON users(tenant_id);
CREATE INDEX idx_users_vippo ON users(vippo_user_id);
```

### wallets
Billetera del usuario. Un usuario tiene 1 wallet por tenant (multi-tenancy).

```sql
CREATE TABLE wallets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  balance DECIMAL(18,4) NOT NULL DEFAULT 0.0000,  -- balance synced
  balance_offline DECIMAL(18,4),                   -- balance local reportado
  pending_sync_count INTEGER DEFAULT 0,
  currency TEXT NOT NULL DEFAULT 'VES',
  status TEXT NOT NULL DEFAULT 'active',          -- active, frozen, blocked
  daily_limit DECIMAL(18,4) DEFAULT 1000.00,
  monthly_limit DECIMAL(18,4) DEFAULT 5000.00,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, tenant_id, currency)
);

CREATE INDEX idx_wallets_user ON wallets(user_id);
CREATE INDEX idx_wallets_tenant ON wallets(tenant_id);
```

### transactions
Registro de cada transacción (cashin, pago, cashout).

```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY,                -- UUID generado en cliente
  wallet_id UUID NOT NULL REFERENCES wallets(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  type TEXT NOT NULL,                 -- cashin, pay, cashout, fee, reversal
  amount DECIMAL(18,4) NOT NULL,
  currency TEXT NOT NULL,
  counterparty_wallet_id UUID REFERENCES wallets(id),
  counterparty_external TEXT,         -- para merchants o bancos
  status TEXT NOT NULL DEFAULT 'pending_sync',
                                     -- pending_sync, synced, reversed, failed
  offline_signature TEXT,             -- ECDSA signature hex
  offline_nonce UUID,                 -- UUID v4 generado en dispositivo
  offline_timestamp TIMESTAMPTZ,      -- cuando se firmó offline
  synced_at TIMESTAMPTZ,
  vippo_cashin_id TEXT,
  vippo_cashout_id TEXT,
  device_id UUID REFERENCES devices(id),
  location JSONB,                     -- { lat, lng } opcional
  metadata JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  CONSTRAINT chk_type CHECK (type IN ('cashin', 'pay', 'cashout', 'fee', 'reversal')),
  CONSTRAINT chk_status CHECK (status IN ('pending_sync', 'synced', 'reversed', 'failed')),
  CONSTRAINT chk_amount_positive CHECK (amount > 0)
);

CREATE UNIQUE INDEX idx_tx_nonce ON transactions(offline_nonce) WHERE offline_nonce IS NOT NULL;
CREATE INDEX idx_tx_wallet_pending ON transactions(wallet_id) WHERE status = 'pending_sync';
CREATE INDEX idx_tx_tenant_created ON transactions(tenant_id, created_at DESC);
CREATE INDEX idx_tx_status ON transactions(status, created_at DESC);
```

### ledger
Append-only. Una transacción genera 2+ entries (debit + credit).

### devices
Dispositivos autorizados por usuario (móvil, tablet POS).

### cashouts
Liquidaciones del tenant hacia su banco.

### audit_log
Log de auditoría de operaciones sensibles.

## Row-Level Security (RLS)

PostgreSQL RLS policies para multi-tenancy:

```sql
-- Habilitar RLS en todas las tablas
ALTER TABLE wallets ENABLE ROW LEVEL SECURITY;
ALTER TABLE transactions ENABLE ROW LEVEL SECURITY;
ALTER TABLE ledger ENABLE ROW LEVEL SECURITY;
ALTER TABLE devices ENABLE ROW LEVEL SECURITY;
ALTER TABLE audit_log ENABLE ROW LEVEL SECURITY;

-- Política: cada query debe filtrar por tenant_id
CREATE POLICY tenant_isolation_wallets ON wallets
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- (similar para las demás tablas)
```

**En el código de Lambda:**
```python
# Antes de cada query, setear el tenant
await conn.execute(f"SET app.current_tenant_id = '{tenant_id}'")
```

## Multi-currency

- Soporte futuro: COP, MXN, USD
- Por ahora: solo VES (bolívar venezolano)
- `wallets.currency` permite escalabilidad

## Auditoría y retención

- `transactions`: 7 años (regulatorio VE)
- `ledger`: 7 años
- `audit_log`: 7 años
- `wallets.balance` snapshots mensuales para auditoría

---

**Próximo documento:** `06-api-contract.md`