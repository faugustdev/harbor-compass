# ViCheck MVP — Requisitos Funcionales

## Contexto

Este documento lista los **requisitos funcionales (RF)** del MVP de ViCheck. Los RF describen QUÉ debe hacer el sistema. Los detalles de implementación están en la arquitectura.

**Convenciones:**
- `RF-XXX-NNN`: Requisito Funcional numerado
- `P0`: Crítico, MVP no puede salir sin esta
- `P1`: Importante, MVP puede salir pero incompleto
- `P2`: Deseable, post-MVP

---

## RF-001: Registro y autenticación

### RF-001-001 [P0] Registro de cliente final
El cliente puede registrarse en ViCheck con su número de teléfono.
- **Entradas:** número de teléfono, nombre completo, cédula, email opcional
- **Proceso:** SMS OTP vía VIPPO + verificación de KYC básico delegado a VIPPO
- **Salidas:** cuenta activa, wallet en estado `pending` esperando CashIn

### RF-001-002 [P0] Onboarding de prestador (comercio)
Un prestador de servicios puede darse de alta como tenant-comercio.
- **Entradas:** datos del comercio (RIF, razón social, dirección, banco, cuenta)
- **Proceso:** VIPPO valida RIF, crea tenant, asigna `vippo_merchant_id`
- **Salidas:** tenant habilitado, POS app vinculada al tenant_id

### RF-001-003 [P0] Login biométrico
El usuario puede habilitar login con huella o PIN de 6 dígitos.
- **Entradas:** biometría o PIN
- **Proceso:** valida contra Keystore del dispositivo
- **Salidas:** sesión local abierta, JWT refrescado

### RF-001-004 [P1] Logout multi-dispositivo
El usuario puede cerrar sesión y revocar acceso en otros dispositivos.
- **Entradas:** sesión actual
- **Proceso:** backend invalida todos los JWT del usuario
- **Salidas:** confirmación, próxima request devuelve 401

---

## RF-002: CashIn (carga de saldo)

### RF-002-001 [P0] CashIn desde banco del cliente
El cliente puede cargar saldo a su billetera ViCheck desde su cuenta bancaria VIPPO.
- **Entradas:** monto, banco origen, cuenta origen
- **Proceso:** POST `/cashin/{amount}` → llama VIPPO `/validacion de pago` → webhook confirma
- **Salidas:** saldo actualizado, transacción tipo `cashin`, estado `synced`

### RF-002-002 [P0] CashIn fallido
Si la transferencia bancaria falla, el saldo NO se acredita y se muestra error.
- **Proceso:** webhook VIPPO `status: failed` → ledger hace reverso → push notification
- **Salidas:** saldo intacto, transacción `cashin` en estado `failed`

### RF-002-003 [P0] CashIn parcial / pendiente
CashIn puede quedar en estado `pending` hasta 60s esperando webhook.
- **Proceso:** GET `/cashin/{id}` retorna estado actual
- **Salidas:** cliente ve `pending` y puede reintentar

### RF-002-004 [P1] CashIn programado / auto-recarga
El cliente puede configurar auto-recarga cuando saldo < umbral.
- **Entradas:** umbral, monto a recargar
- **Proceso:** backend auto-ejecuta CashIn cuando se cumple condición
- **Salidas:** recarga automática, push notification

---

## RF-003: Pago offline (cliente → comercio)

### RF-003-001 [P0] Pago NFC (chip-to-phone)
El cliente puede pagar acercando su pulsera/tarjeta/celular al POS del comercio.
- **Entradas:** monto, comercio destino
- **Proceso:**
  1. Cliente acerca tag NFC al POS
  2. POS lee UID del tag
  3. POS consulta balance local del UID
  4. Cliente confirma con PIN si monto ≥ umbral_tenant
  5. POS firma transacción (ECDSA)
  6. POS descuenta del balance local, append a vault
  7. POS notifica OK al comercio
- **Salidas:** transacción en vault local, estado `pending_sync`
- **Latencia objetivo:** < 300 ms

### RF-003-002 [P0] Pago dChip QR (phone-to-phone)
El cliente puede pagar escaneando el QR dinámico generado por el POS.
- **Entradas:** monto
- **Proceso:**
  1. POS genera QR cada 30s (HMAC + nonce + ttl)
  2. Cliente escanea QR
  3. Cliente verifica HMAC + ttl
  4. Cliente firma transacción
  5. Cliente transmite tx firmada a POS vía canal local (BLE/WiFi Direct/NFC)
  6. POS verifica, descuenta local
- **Salidas:** tx en vault local del POS

### RF-003-003 [P0] Pago phone-to-phone P2P
Un cliente puede transferir saldo a otro cliente sin POS comercial.
- **Entradas:** destinatario (teléfono o UID), monto
- **Proceso:** misma lógica, no requiere POS intermediario
- **Salidas:** saldo transferido entre wallets

### RF-003-004 [P0] Confirmación con PIN
Para montos ≥ umbral configurable por tenant, el cliente debe ingresar PIN.
- **Proceso:** sin PIN, la tx se rechaza
- **Configuración:** cada tenant define su umbral (ej: $5 USD)

### RF-003-005 [P0] Validación de saldo suficiente
Si el cliente no tiene saldo suficiente, la transacción se rechaza.
- **Proceso:** comparación local antes de firmar
- **Salidas:** mensaje "saldo insuficiente" al cliente

### RF-003-006 [P1] Límite diario de transacciones
Cada wallet tiene un límite diario configurable por el backend.
- **Salidas:** tx rechazadas si exceden el límite

### RF-003-007 [P0] Recibo de transacción offline
El cliente recibe confirmación visual inmediata de la transacción.
- **Salidas:** pantalla con monto, fecha, comercio (si aplica), nuevo balance

---

## RF-004: Sync online (transacciones offline → backend)

### RF-004-001 [P0] Detección automática de conectividad
El dispositivo detecta cuando vuelve internet (NetworkCallback).
- **Proceso:** NetworkCapabilities.NET_CAPABILITY_VALIDATED
- **Salidas:** trigger automático de sync

### RF-004-002 [P0] Batch sync de transacciones pendientes
El dispositivo sincroniza todas las transacciones pendientes en batch.
- **Entradas:** batch de tx firmadas localmente
- **Proceso:** POST `/sync` con batch
- **Salidas:** respuesta con resultado por tx (aceptada/duplicada/rechazada)

### RF-004-003 [P0] Idempotencia de sync
Una transacción sincronizada dos veces no causa doble descuento.
- **Proceso:** backend valida `tx_id` único (UUID)
- **Salidas:** tx duplicada marcada como `already_synced` en respuesta

### RF-004-004 [P0] Reintento con backoff exponencial
Si sync falla, el dispositivo reintenta con backoff (1s, 2s, 4s, 8s, 16s).
- **Proceso:** máximo 5 reintentos antes de marcar como `failed`
- **Salidas:** tx marcadas como `failed` requieren intervención manual

### RF-004-005 [P1] Sync parcial (chunked)
Si hay >100 tx pendientes, sync en lotes para evitar timeout.
- **Proceso:** chunk size = 100, hasta 10 chunks paralelos
- **Salidas:** todos los chunks subidos en ≤30s para 1000 tx

### RF-004-006 [P0] Cola persistente
La cola de tx pendientes sobrevive cierre de app, batería baja, reinicio.
- **Proceso:** vault local en disco (no RAM)
- **Salidas:** tx no se pierden

---

## RF-005: CashOut (liquidación)

### RF-005-001 [P0] CashOut de comercio a banco
El comercio puede transferir su saldo disponible a su cuenta bancaria registrada.
- **Entradas:** monto a liquidar
- **Proceso:** POST `/cashout` → VIPPO `/liquidacion` → webhook confirma
- **Salidas:** saldo descontado, tx `cashout` completada, fondos en banco

### RF-005-002 [P0] CashOut programado / automático
El sistema puede liquidar automáticamente cuando el saldo acumulado > umbral.
- **Entradas:** umbral configurable por tenant
- **Proceso:** CRON backend evalúa diariamente
- **Salidas:** liquidación automática

### RF-005-003 [P0] Reporte de liquidación
El comercio recibe reporte de liquidaciones.
- **Salidas:** dashboard con histórico, próximo CashOut programado, balance retenido

### RF-005-004 [P1] CashOut parcial
El comercio puede elegir cuánto liquidar (no todo).
- **Entradas:** monto o "todo"
- **Salidas:** el resto queda en balance del tenant

---

## RF-006: Multi-tenancy

### RF-006-001 [P0] Aislamiento de datos por tenant
Cada tenant solo ve sus propios usuarios, transacciones y balances.
- **Proceso:** RLS en Postgres, `tenant_id` en todas las queries
- **Salidas:** tests E2E validan aislamiento

### RF-006-002 [P0] Branding por tenant
Cada tenant puede tener su logo, color y nombre en la app del cliente.
- **Entradas:** tenant config (logo, color primario, nombre comercial)
- **Salidas:** UI personalizada cuando cliente interactúa con ese tenant

### RF-006-003 [P1] Configuración de fees por tenant
Cada tenant puede tener fee distinto (porcentaje sobre cada tx).
- **Entradas:** fee_percentage
- **Proceso:** backend calcula fee en cada tx, lo asigna a tenant VIPPO
- **Salidas:** reporte de fees acumulados

---

## RF-007: Reportes y dashboard

### RF-007-001 [P0] Balance actual
El usuario ve su balance actual en la home de la app.
- **Entradas:** sesión del usuario
- **Salidas:** balance actualizado (local + remote)

### RF-007-002 [P0] Histórico de transacciones
El usuario ve sus últimas 100 transacciones.
- **Entradas:** usuario autenticado
- **Salidas:** lista paginada de tx con estado

### RF-007-003 [P1] Filtros y búsqueda de tx
El usuario puede filtrar por fecha, monto, tipo.
- **Salidas:** lista filtrada

### RF-007-004 [P0] Reporte comercio (POS)
El POS comercio muestra ventas del día, semana, mes.
- **Entradas:** tenant_id, fecha
- **Salidas:** reporte agregado

---

## RF-008: dChip (QR dinámico)

### RF-008-001 [P0] Generación de dChip
El POS genera un QR que se renueva cada 30 segundos.
- **Entradas:** tenant_id, device_id
- **Proceso:** HMAC-SHA256(tenant_id, device_id, timestamp, nonce) codificado en base64 + JSON
- **Salidas:** QR con TTL=30s

### RF-008-002 [P0] Validación de dChip
El cliente verifica HMAC + ttl antes de aceptar QR.
- **Proceso:** comparación HMAC, verificación ttl no expirado
- **Salidas:** tx firmada y enviada al POS si QR válido

### RF-008-003 [P1] dChip con firma ECDSA
Para mayor seguridad, dChip usa ECDSA en vez de HMAC.
- **Proceso:** POS firma con llave privada, cliente verifica con llave pública del tenant
- **Salidas:** seguridad contra POS comprometido

---

## RF-009: Seguridad y fraude

### RF-009-001 [P0] Firmas ECDSA en cada transacción offline
Cada tx offline va firmada con llave privada del dispositivo del pagador.
- **Proceso:** ECDSA P-256, hash SHA-256 del payload
- **Salidas:** backend puede verificar firma contra llave pública

### RF-009-002 [P0] Nonce único por transacción
Cada tx tiene un UUID único que previene duplicación.
- **Proceso:** UUID v4 generado en el dispositivo
- **Salidas:** backend rechaza tx con UUID duplicado

### RF-009-003 [P0] Rate limiting por wallet
Cada wallet tiene un máximo de N tx por minuto/hora/día.
- **Salidas:** tx que exceden el límite se rechazan

### RF-009-004 [P1] Anomaly detection (reglas básicas)
Backend detecta comportamientos sospechosos (montos atípicos, frecuencia inusual).
- **Salidas:** tx marcada para revisión manual

### RF-009-005 [P1] Bloqueo de wallet
El usuario puede bloquear su wallet desde la app o por soporte.
- **Entradas:** acción del usuario o regla de fraude
- **Salidas:** wallet en estado `blocked`, no permite tx

### RF-009-006 [P0] Cierre de sesión seguro
Logout borra JWT del dispositivo y limpia llaves en RAM.
- **Salidas:** dispositivo queda en estado neutro

---

## RF-010: NFC

### RF-010-001 [P0] Lectura de UID NFC
El POS puede leer el UID de cualquier tag NFC compatible (NTAG213/215/216, MIFARE).
- **Entradas:** tag NFC acercado
- **Proceso:** Android NFC API, Foreground Dispatch
- **Salidas:** UID de 7-10 bytes en hex

### RF-010-002 [P0] Emparejado de tag NFC con wallet
El cliente puede emparejar un tag NFC físico con su wallet ViCheck.
- **Entradas:** tag NFC + PIN del usuario
- **Proceso:** backend asocia UID → wallet_id
- **Salidas:** tag físico ahora representa la wallet del cliente

### RF-010-003 [P1] Soporte multi-tag
El cliente puede tener múltiples tags NFC emparejados con la misma wallet.
- **Entradas:** múltiples tags NFC
- **Proceso:** backend almacena N UID → mismo wallet_id
- **Salidas:** todos los tags funcionan como identificador

### RF-010-004 [P1] Pulseras NTAG216
Soporte específico para pulseras NFC NTAG216 (888 bytes de memoria).
- **Uso futuro:** guardar vCard o configuración offline en el chip

---

## RF-011: Notificaciones

### RF-011-001 [P0] Push de tx recibida
El cliente recibe push cuando reciba un pago.
- **Salidas:** push "Recibiste $X de Y"

### RF-011-002 [P0] Push de CashIn confirmado
El cliente recibe push cuando su carga de saldo fue exitosa.
- **Salidas:** push "Saldo acreditado: $X"

### RF-011-003 [P1] Push de sync completado
El comercio recibe push cuando se completa el sync de tx pendientes.
- **Salidas:** push "X transacciones sincronizadas"

### RF-011-004 [P1] Push de CashOut completado
El comercio recibe push cuando su liquidación es exitosa.
- **Salidas:** push "Liquidación de $X completada"

---

## RF-012: Administración (interno VIPPO)

### RF-012-001 [P0] Admin dashboard
VIPPO admin puede ver dashboard con métricas globales.
- **Salidas:** dashboard con tx/min, tenures activos, balances totales, alertas

### RF-012-002 [P0] Onboarding de tenant
Admin puede crear tenants nuevos con datos básicos.
- **Salidas:** tenant activo

### RF-012-003 [P1] Auditoría de tx individuales
Admin puede buscar cualquier tx por ID y ver detalles completos.
- **Salidas:** vista detallada de tx + firmas + signatures

### RF-012-004 [P1] Reconciliación manual
Admin puede ejecutar reconciliación manual para resolver discrepancias.
- **Entradas:** rango de fechas, tenant_id
- **Salidas:** reporte de diferencias y acciones

---

## Resumen de prioridades

| Prioridad | Cantidad | % |
|---|---|---|
| P0 (crítico MVP) | 32 | 64% |
| P1 (importante) | 14 | 28% |
| P2 (post-MVP) | 4 | 8% |
| **Total RF** | **50** | **100%** |

**Conclusión MVP:** 32 requisitos P0 son necesarios para salir a closed beta.

---

**Próximo documento:** `04-requirements/non-functional.md` — requisitos no funcionales (seguridad, performance, compliance).