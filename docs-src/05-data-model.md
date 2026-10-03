# ViCheck MVP — Modelo de Datos (PostgreSQL) — Modelo corregido

> **IMPORTANTE:** ViCheck NO custodia dinero. El "balance" en `wallets` es un **crédito lógico** contra el Aliado. El dinero real está en cuentas bancarias externas (del cliente y del aliado).

## Diagrama ER simplificado

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│   tenants    │       │    users     │       │   wallets    │
├──────────────┤       ├──────────────┤       ├──────────────┤
│ id (PK)      │◄──┐   │ id (PK)      │◄──┐   │ id (PK)      │
│ name         │   │   │ tenant_id    │   │   │ user_id (FK) │─┘
│ slug         │   │   │ phone        │   │   │ tenant_id (FK)│
│ vippo_merchant│  │   │ email        │   │   │ balance      │ ← saldo LÓGICO
│ bank_account │   │   │ full_name    │   │   │ offline_pending│
│ status       │   │   │ kyc_status   │   │   │ offline_max  │
│ branding     │   │   │ vippo_user_id│   │   │ currency     │
└──────────────┘│  │  └──────────────┘   │   │ status       │
                │  └─────────────────────┘   │ daily_limit  │
                │                            │ monthly_limit│
                │                            └──────┬───────┘
                │                                   │
                │                                   ▼
                │                            ┌──────────────┐
                │                            │ transactions │
                │                            ├──────────────┤
                │                            │ id (PK)      │
                │                            │ wallet_id(FK)│
                │                            │ tenant_id(FK)│
                │                            │ type         │
                │                            │ amount       │
                │                            │ counterparty_aliado│
                │                            │ status       │
                │                            │ offline_sig  │
                │                            │ nonce        │
                │                            │ vippo_cashin_id│
                │                            │ vippo_transfer_id│
                │                            └──────┬───────┘
                │                                   │
                │                                   ▼
                │                            ┌──────────────┐
                │                            │  sync_log    │
                │                            ├──────────────┤
                │                            │ id (PK)      │
                │                            │ device_id    │
                │                            │ user_id      │
                │                            │ batch_id     │
                │                            │ tx_count     │
                │                            │ status       │
                │                            └──────────────┘

┌──────────────┐       ┌──────────────┐
│  transfers   │       │  devices     │
├──────────────┤       ├──────────────┤
│ id (PK)      │       │ id (PK)      │
│ from_aliado  │       │ user_id (FK) │
│ to_aliado    │       │ tenant_id    │
│ user_id      │       │ device_type  │
│ amount       │       │ public_key   │
│ status       │       │ last_seen_at │
│ vippo_id     │       │ status       │
│ created_at   │       └──────────────┘
└──────────────┘
```

## Tablas detalladas

### tenants
Representa cada aliado/comercio afiliado a ViCheck.

```sql
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  vippo_merchant_id TEXT UNIQUE,
  vippo_bank_account_id TEXT,
  fee_percentage DECIMAL(5,4) DEFAULT 0.0050,  -- 0.5%
  pin_required_threshold DECIMAL(18,4) DEFAULT 5.00,
  status TEXT NOT NULL DEFAULT 'active',
  branding JSONB,
  config JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### users
Usuarios finales (clientes) que usan ViCheck como wallet lógica.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  phone TEXT UNIQUE NOT NULL,
  email TEXT,
  full_name TEXT NOT NULL,
  cedula TEXT,
  kyc_status TEXT NOT NULL DEFAULT 'pending',
  vippo_user_id TEXT UNIQUE,
  pin_hash TEXT,
  biometric_enabled BOOLEAN DEFAULT false,
  status TEXT NOT NULL DEFAULT 'active',
  language TEXT DEFAULT 'es',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### wallets
**Wallet LÓGICA del cliente, NO dinero real**. Es un crédito contra el Aliado.

```sql
CREATE TABLE wallets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  balance DECIMAL(18,4) NOT NULL DEFAULT 0.0000,  -- saldo LÓGICO
  currency TEXT NOT NULL DEFAULT 'VES',
  status TEXT NOT NULL DEFAULT 'active',
  daily_limit DECIMAL(18,4) DEFAULT 1000.00,
  monthly_limit DECIMAL(18,4) DEFAULT 5000.00,
  offline_pending DECIMAL(18,4) DEFAULT 0,
  offline_max_amount DECIMAL(18,4) DEFAULT 500.00,  -- tope configurado por backend
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, tenant_id, currency)
);
```

### transactions
Registro de cada transacción (cashin, pago, transferencia).

```sql
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

CREATE UNIQUE INDEX idx_tx_offline_nonce ON transactions(offline_nonce) WHERE offline_nonce IS NOT NULL;
CREATE INDEX idx_tx_wallet_pending ON transactions(wallet_id) WHERE status = 'pending_sync';
CREATE INDEX idx_tx_tenant_created ON transactions(tenant_id, created_at DESC);
```

### sync_log
Log append-only de cada sync realizado.

```sql
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
```

### transfers
Transferencias de saldo entre aliados (cuando cliente gasta crédito con aliado A en aliado B).

```sql
CREATE TABLE transfers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  from_aliado_id UUID NOT NULL REFERENCES tenants(id),
  to_aliado_id UUID NOT NULL REFERENCES tenants(id),
  user_id UUID NOT NULL REFERENCES users(id),
  amount DECIMAL(18,4) NOT NULL,
  currency TEXT NOT NULL DEFAULT 'VES',
  status TEXT NOT NULL DEFAULT 'pending',
  vippo_transfer_id TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  completed_at TIMESTAMPTZ,
  metadata JSONB
);

CREATE INDEX idx_transfers_user ON transfers(user_id, created_at DESC);
```

### devices
Dispositivos autorizados por usuario (móvil).

```sql
CREATE TABLE devices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  device_type TEXT,
  public_key TEXT NOT NULL,
  device_name TEXT,
  status TEXT NOT NULL DEFAULT 'active',
  last_seen_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## Multi-tenancy (RLS)

```sql
ALTER TABLE wallets ENABLE ROW LEVEL SECURITY;
ALTER TABLE transactions ENABLE ROW LEVEL SECURITY;
ALTER TABLE sync_log ENABLE ROW LEVEL SECURITY;
ALTER TABLE devices ENABLE ROW LEVEL SECURITY;
ALTER TABLE transfers ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_wallets ON wallets
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
-- (similar para las demás tablas)
```

## Auditoría y retención

- `transactions`: 7 años (regulatorio)
- `sync_log`: 7 años
- `transfers`: 7 años
- `wallets.balance` snapshots mensuales para auditoría

---

**Próximo documento:** `06-api-contract.md`