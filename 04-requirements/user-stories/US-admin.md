# Historias de Usuario: Admin VIPPO

> Admin = personal interno VIPPO que gestiona tenants, monitorea el sistema y resuelve incidencias.

---

## US-A-001 — Autenticación

**HU-300 [P0]** Como admin, quiero iniciar sesión con credenciales de VIPPO + 2FA, para tener acceso seguro.

**Criterios de aceptación:**
- [ ] Email + password
- [ ] 2FA vía TOTP (Google Authenticator)
- [ ] Sesión expira en 8h

---

**HU-301 [P0]** Como admin, quiero ver mis permisos asignados, para saber qué puedo hacer.

**Criterios de aceptación:**
- [ ] Lista de permisos en mi perfil
- [ ] Denegación clara de acciones no permitidas

---

## US-A-002 — Tenant management

**HU-310 [P0]** Como admin, quiero ver la lista de todos los tenants, para gestión general.

**Criterios de aceptación:**
- [ ] Tabla con filtros (estado, fecha, búsqueda por nombre/RIF)
- [ ] Paginación
- [ ] Acciones: ver detalle, editar, deshabilitar

---

**HU-311 [P0]** Como admin, quiero crear un nuevo tenant, para incorporar nuevos comercios.

**Criterios de aceptación:**
- [ ] Formulario con todos los campos
- [ ] Validación de RIF y banco
- [ ] Generación automática de tenant_id

---

**HU-312 [P0]** Como admin, quiero ver el detalle de un tenant, para entender su operación.

**Criterios de aceptación:**
- [ ] Info general: RIF, banco, contacto
- [ ] Métricas: ventas, usuarios, balance
- [ ] Histórico de tx

---

**HU-313 [P1]** Como admin, quiero deshabilitar un tenant, para bloqueo temporal o definitivo.

**Criterios de aceptación:**
- [ ] Acción "Deshabilitar" en detalle
- [ ] Confirmación con razón
- [ ] Tenant queda inactivo pero sus datos se conservan

---

## US-A-003 — Monitoreo

**HU-320 [P0]** Como admin, quiero ver métricas globales del sistema en tiempo real, para entender el estado.

**Criterios de aceptación:**
- [ ] Dashboard con:
  - Tx por minuto
  - Tx totales hoy
  - Volumen transado hoy
  - CashOut realizados hoy
  - Errores / fallos
  - Latencia promedio

---

**HU-321 [P0]** Como admin, quiero ver alertas activas, para tomar acción.

**Criterios de aceptación:**
- [ ] Banner de alertas en dashboard
- [ ] Lista de alertas activas
- [ ] Acciones: ver detalle, escalar, marcar como resuelta

---

**HU-322 [P0]** Como admin, quiero ver logs de auditoría, para investigar incidentes.

**Criterios de aceptación:**
- [ ] Búsqueda por tenant, usuario, fecha, acción
- [ ] Vista detallada con contexto

---

## US-A-004 — Anti-fraude

**HU-330 [P0]** Como admin, quiero ver tx marcadas como sospechosas por el motor de anomalías.

**Criterios de aceptación:**
- [ ] Cola de revisión con tx marcadas
- [ ] Detalle de cada tx + razón de la alerta
- [ ] Acciones: aprobar, rechazar, escalar

---

**HU-331 [P0]** Como admin, quiero congelar una wallet, para evitar pérdidas mientras se investiga.

**Criterios de aceptación:**
- [ ] Acción "Congelar wallet" desde detalle de tx o wallet
- [ ] Razón obligatoria
- [ ] Wallet queda en estado `frozen`, no permite tx

---

**HU-332 [P1]** Como admin, quiero reversar una transacción fraudulenta, para recuperar fondos.

**Criterios de aceptación:**
- [ ] Acción "Reversar tx" desde detalle
- [ ] Crea tx de reverso en ledger
- [ ] Saldo se restaura

---

**HU-333 [P1]** Como admin, quiero ver patrones de fraude (múltiples tx de un mismo UID en poco tiempo).

**Criterios de aceptación:**
- [ ] Vista de "Actividad sospechosa"
- [ ] Lista de UIDs con patrones atípicos

---

## US-A-005 — Conciliación

**HU-340 [P0]** Como admin, quiero ejecutar reconciliación diaria, para cuadrar tx offline con backend.

**Criterios de aceptación:**
- [ ] Job automático a las 3am (configurable)
- [ ] Detecta discrepancias
- [ ] Reporte de status

---

**HU-341 [P0]** Como admin, quiero ver reporte de reconciliación, para identificar problemas.

**Criterios de aceptación:**
- [ ] Lista de tx con discrepancia
- [ ] Detalle: esperado vs real
- [ ] Acciones: marcar, escalar

---

**HU-342 [P1]** Como admin, quiero ejecutar reconciliación manual, para casos específicos.

**Criterios de aceptación:**
- [ ] Botón "Reconciliar ahora"
- [ ] Selección de rango y tenant
- [ ] Output en pantalla

---

## US-A-006 — Configuración global

**HU-350 [P0]** Como admin, quiero configurar el fee por defecto por transacción, para nuevos tenants.

**Criterios de aceptación:**
- [ ] Settings globales → Fee por defecto
- [ ] Aplica a tenants nuevos
- [ ] Cambios requieren aprobación

---

**HU-351 [P0]** Como admin, quiero configurar límites por defecto (CashIn, tx diaria, etc.).

**Criterios de aceptación:**
- [ ] Settings globales → Límites
- [ ] Por defecto para nuevos tenants
- [ ] Configurable por tenant

---

**HU-352 [P1]** Como admin, quiero programar mantenimiento, para informar a usuarios.

**Criterios de aceptación:**
- [ ] Calendario de mantenimientos
- [ ] Banner automático en app
- [ ] Push notification

---

## US-A-007 — Reportes regulatorios

**HU-360 [P0]** Como admin, quiero generar reporte de operaciones para SUDEBAN, para cumplimiento regulatorio.

**Criterios de aceptación:**
- [ ] Formato compatible con SUDEBAN
- [ ] Generación por rango de fechas
- [ ] Incluye todas las operaciones del período

---

**HU-361 [P0]** Como admin, quiero generar reporte fiscal para SENIAT, para declaración de impuestos.

**Criterios de aceptación:**
- [ ] Formato compatible con SENIAT
- [ ] Desglose por tipo de operación
- [ ] Descargable en PDF/CSV

---

**HU-362 [P1]** Como admin, quiero auditar accesos a datos personales, para cumplimiento LOPDP.

**Criterios de aceptación:**
- [ ] Log de quién accedió a qué dato personal y cuándo
- [ ] Reporte auditable

---

## Resumen

| Historia | P0 | P1 |
|---|---|---|
| Autenticación | 2 | 0 |
| Tenant mgmt | 3 | 1 |
| Monitoreo | 3 | 0 |
| Anti-fraude | 2 | 2 |
| Conciliación | 2 | 1 |
| Configuración | 2 | 1 |
| Regulatorio | 2 | 1 |
| **Total admin** | **16** | **6** |

---

**Próximo documento:** `use-cases/UC-01-cashin.md` — caso de uso detallado de CashIn.