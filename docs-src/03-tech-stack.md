# ViCheck MVP — Stack Tecnológico y Justificación

## 1. Stack cliente (móvil)

### 1.1 Kotlin Multiplatform (KMP) — **RECOMENDADO**

**Por qué:** KMP permite compartir la **lógica de negocio** (core engine, offline-first, crypto, sync) entre Android e iOS mientras mantiene UI nativa. Es el patrón que usa Cash App, Block, Mercado Pago, Binance.

| Capa | Tecnología | Razón |
|---|---|---|
| Shared core | Kotlin Multiplatform | Lógica offline-first, crypto, modelos |
| APP Android | KMP | mismo shared core |
| APP iOS (futuro) | KMP | mismo shared core |
| UI Android | Jetpack Compose | Estándar Android moderno |
| UI iOS (futuro) | SwiftUI | requerido para conversión |

**Ventajas:**
- Un solo lugar para la lógica de vault, ECDSA, sync
- Si en el futuro se quiere iOS, no se duplica código
- Mejor testeo: shared core = tests en una sola plataforma
- Reducción de bugs (lógica en una sola implementación)

**Desventajas:**
- Curva de aprendizaje inicial
- Algunas APIs Android-específicas (Foreground Dispatch) requieren expect/actual
- Build times ligeramente mayores

**Decisión:** ✅ **KMP con Compose Multiplatform** para shared core, **Jetpack Compose** para UI Android.

### 1.2 Alternativas consideradas

| Alternativa | Por qué descartada |
|---|---|
| **Flutter** | Dart no es Kotlin,团队的 equipo es Kotlin, KMP reusa skills existentes |
| **React Native** | Bridge JS-Native agrega latencia, peor para crypto de baja latencia |
| **Native Android solo** | Si en MVP no hay iOS, vale la pena. Pero KMP no cuesta más |
| **Xamarin** | Ecosistema en declive, C# no es Kotlin |

## 2. Stack Android (capa plataforma)

### 2.1 Lenguaje y frameworks

| Componente | Tecnología | Versión | Justificación |
|---|---|---|---|
| Lenguaje | Kotlin | 2.0+ | Estándar Android, KMP-compatible |
| UI | Jetpack Compose | 1.7+ | Estándar moderno, declarativo |
| DI | Hilt | 2.50+ | Estándar Google, KSP-based |
| Async | Coroutines + Flow | 1.8+ | Estándar Kotlin async |
| Navigation | Compose Navigation | 2.8+ | Integración Compose |
| Build | Gradle + KSP | 8.7+ | Build moderno |
| Min SDK | API 24 (Android 7.0) | — | Cubre >97% mercado VE |
| Target SDK | API 35 (Android 15) | — | Play Store requirement |

### 2.2 Librerías clave

| Librería | Propósito | Licencia |
|---|---|---|
| **Room** | ORM SQLite | Apache 2 |
| **SQLCipher** | Encriptación DB | BSD |
| **Tink** | Criptografía (Google) | Apache 2 |
| **Bouncy Castle** | ECDSA, X.509 | MIT |
| **Retrofit + OkHttp** | HTTP client | Apache 2 |
| **Moshi** | JSON serialization | Apache 2 |
| **Kotlinx Serialization** | Alternativa Moshi | Apache 2 |
| **WorkManager** | Background sync | Apache 2 |
| **DataStore** | Preferences encriptadas | Apache 2 |
| **ZXing** | QR scanner/generator | Apache 2 |
| **CameraX** | QR scanning | Apache 2 |
| **Firebase Crashlytics** | Crash reporting | Free tier |
| **Firebase Analytics** | Analytics | Free tier |

## 3. Stack Backend

### 3.1 AWS SAM + Serverless

VIPPO ya usa AWS SAM (ver `redesigned-couscous` en el monorepo). Mantener consistencia con su stack existente.

| Componente | Tecnología | Justificación |
|---|---|---|
| Lenguaje | Python 3.14 | Coherente con redesigned-couscous |
| Framework API | FastAPI sobre Lambda | Tipado fuerte, async, OpenAPI automático |
| Orquestación | AWS SAM | Ya en uso en VIPPO |
| Compute | AWS Lambda | Serverless, escala automática |
| API Gateway | REST API | Costo bajo, integración directa con Lambda |
| Async | Lambda + SQS + EventBridge | Decouple processing |
| ORM | SQLAlchemy 2.0 async | Estándar Python |
| Migrations | Alembic | Estándar SQLAlchemy |
| Validación | Pydantic v2 | Estándar FastAPI |
| Crypto | cryptography (PyCA) | Estándar, audited |
| HTTP client | httpx | Async HTTP |
| Tests | pytest + pytest-asyncio | Estándar Python |
| Lint | ruff | Rápido, moderno |
| Format | black | Estándar Python |

### 3.2 Bases de datos

| DB | Uso | Versión | Justificación |
|---|---|---|---|
| **PostgreSQL** | Ledger principal | 16 | ACID, append-only natural, jsonb |
| **Redis** | Cache balances + rate limit | 7 | Latencia <1ms, hot path |
| Hosted | RDS + ElastiCache | — | Managed, backups automáticos |

**Multi-tenancy:** columna `tenant_id` en todas las tablas + row-level security policies (RLS) de Postgres.

### 3.3 Almacenamiento de llaves criptográficas | Servicio | Uso |
|---|---|
| AWS KMS | Llave maestra del backend |
| AWS Secrets Manager | API keys de VIPPO, bank credentials |
| Android Keystore | Llaves ECDSA del dispositivo |

## 4. Stack de infraestructura

| Capa | Tecnología | Justificación |
|---|---|---|
| Cloud | AWS | Ya en uso en VIPPO |
| IaC | AWS SAM + CDK (futuro) | SAM ahora es lo que es |
| CI/CD | GitHub Actions | Free para private repos, ecosystem |
| Observabilidad | CloudWatch + X-Ray | Nativo AWS |
| Secrets | AWS Secrets Manager | Nativo, rotation |
| Logs | CloudWatch Logs | Nativo |
| Monitoring | CloudWatch Alarms | Básico, suficiente para MVP |
| Analytics mobile | Firebase Analytics | Free tier |
| Crash reporting | Firebase Crashlytics | Free tier |
| Distribution (Play Store) | Fastlane | Estándar mobile |

## 5. Decisiones clave del MVP

### 🔴 D-001: Kotlin Multiplatform para shared core
- **Status:** Aprobado para MVP
- **Contexto:** MVP Android, posible iOS futuro
- **Decisión:** Shared core en KMP + Jetpack Compose Android
- **Consecuencias:** +1 día setup KMP, ahorro futuro Si iOS

### 🔴 D-002: Backend en AWS SAM + Lambda Python 3.14
- **Status:** Aprobado
- **Razón:** Coherencia con stack VIPPO existente

### 🔴 D-003: PostgreSQL hosted en RDS
- **Status:** Aprobado
- **Razón:** ACID, jsonb, RLS para multi-tenancy

### 🔴 D-004: Criptografía con Tink + Bouncy Castle
- **Status:** Aprobado
- **Razón:** Estándar Google + audited

### 🔴 D-005: Multi-tenancy via RLS de Postgres
- **Status:** Aprobado
- **Razón:** Aislamiento fuerte, una sola DB

### 🟡 D-006: API Gateway REST (no HTTP API)
- **Status:** Pendiente validación
- **Razón:** REST API soporta mejor integraciones, API keys usage plans
- **Alternativa:** HTTP API es 70% más barato

### 🟡 D-007: dChip QR via HMAC + nonce
- **Status:** Pendiente validación
- **Razón:** Simple, suficiente para MVP
- **Alternativa:** ECDSA firmado por POS (más seguro, +complejidad)

### 🟡 D-008: Sin iOS en MVP
- **Status:** Aprobado por restricción de recursos
- **Razón:** KMP permite agregar después sin reescribir lógica

### 📡 D-009: Comunicación sin internet entre dispositivos
- **Status:** Pendiente validación
- **Opciones:** NFC peer-to-peer, WiFi Direct, BLE proximity
- **Para MVP:** asumimos que NFC + BLE + dQR próximos son suficientes

---

## 6. Stack que se descarta por decisión

| Stack | Por qué se descarta |
|---|---|
| ~~Spring Boot / Java~~ | Python consistente con VIPPO |
| ~~MongoDB~~ | Postgres para ledger ACID |
| ~~DynamoDB~~ | Postgres para queries + RLS |
| ~~Firebase Realtime DB~~ | NoSQL no apropiado para ledger |
| ~~AWS Cognito~~ | KYC manejado por VIPPO |
| ~~Google Pay / Apple Pay~~ | Out of scope MVP |
| ~~Pusher / Ably~~ | WebSocket propio en API Gateway |
| ~~Snowplow~~ | Firebase Analytics suficiente MVP |
| ~~Twilio Verify~~ | OTP via VIPPO existente |

---

## 7. Riesgos técnicos del stack

| Riesgo | Mitigación |
|---|---|
| **KMP con libs específicas Android** | expect/actual para NFC, BLE |
| **Cold start de Lambda** | Provisioned concurrency si latencia importa |
| **Sync conflict offline** | Vector clocks + ECDSA firmas |
| **SQLite local se corrompe** | Backups automáticos + WAL mode |
| **Llaves ECDSA perdidas** | Recovery via VIPPO backend |
| **Multi-tenancy fuga** | Tests E2E + RLS policies + auditoría |

---

**Próximo documento:** `04-requirements/functional-requirements.md` — requisitos funcionales detallados (RF-001, RF-002, ...).