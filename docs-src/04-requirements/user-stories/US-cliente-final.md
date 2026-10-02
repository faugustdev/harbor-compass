# Historias de Usuario: Cliente Final

> Cliente = pasajero / consumidor que carga saldo y paga con NFC o dChip QR.

---

## US-C-001 — Onboarding

**HU-001 [P0]** Como nuevo usuario, quiero registrarme con mi número de teléfono, para empezar a usar ViCheck.

**Criterios de aceptación:**
- [ ] El usuario puede ingresar su número de teléfono venezolano
- [ ] Recibe un código SMS de 6 dígitos
- [ ] Ingresa el código y queda verificado
- [ ] Completa sus datos (nombre, cédula, email opcional)
- [ ] Su wallet es creada en estado `pending`

---

**HU-002 [P0]** Como usuario nuevo, quiero entender qué es ViCheck en 3 pasos, para saber cómo funciona antes de registrarme.

**Criterios de aceptación:**
- [ ] 3 pantallas con ilustraciones
- [ ] Slide 1: "Sin señal. Ni problema."
- [ ] Slide 2: "Acerca y listo"
- [ ] Slide 3: "Seguro siempre"
- [ ] Botón "Empezar" lleva al login

---

**HU-003 [P0]** Como usuario, quiero habilitar biometría o PIN para abrir la app, para que nadie más pueda usar mi billetera.

**Criterios de aceptación:**
- [ ] Después del primer login, se le ofrece configurar biometría
- [ ] Alternativa: PIN de 6 dígitos
- [ ] Si falla la autenticación 5 veces, se bloquea por 5 minutos

---

## US-C-002 — CashIn

**HU-010 [P0]** Como cliente, quiero recargar saldo desde mi banco VIPPO, para tener dinero disponible en ViCheck.

**Criterios de aceptación:**
- [ ] Puede elegir monto predefinido o custom
- [ ] Selecciona banco origen y cuenta
- [ ] Ve resumen del estado
- [ ] Recibe confirmación push cuando se acredita

---

**HU-011 [P0]** Como cliente, quiero ver el estado de mi CashIn si tarda, para saber si se procesó o no.

**Criterios de aceptación:**
- [ ] Pantalla "Procesando..." con animación
- [ ] Si pasan 30s, opción de "Verificar estado"
- [ ] Si falla, mensaje explicativo + opción de reintentar

---

**HU-012 [P1]** Como cliente, quiero configurar auto-recarga cuando saldo < umbral, para no quedarme sin dinero.

**Criterios de aceptación:**
- [ ] Settings → Auto-recarga
- [ ] Configurable: umbral, monto a recargar
- [ ] El sistema ejecuta CashIn automático cuando se cumple

---

## US-C-003 — Pago

**HU-020 [P0]** Como cliente, quiero pagar acercando mi pulsera o tarjeta NFC al POS del comercio, para pagar rápido sin sacar efectivo.

**Criterios de aceptación:**
- [ ] Pantalla "Acerca tu tag al POS"
- [ ] Animación NFC cuando detecta tag
- [ ] El POS lee UID y consulta balance
- [ ] Si monto ≥ umbral, el cliente ingresa PIN
- [ ] Recibe confirmación visual inmediata

---

**HU-021 [P0]** Como cliente, quiero ver el detalle de la transacción, para confirmar que el monto y comercio son correctos.

**Criterios de aceptación:**
- [ ] Pantalla muestra: monto, comercio destino, nuevo balance
- [ ] Botón "Confirmar" requiere PIN
- [ ] Botón "Cancelar" aborta la tx

---

**HU-022 [P0]** Como cliente, quiero ver un recibo digital de mi pago, para tener constancia.

**Criterios de aceptación:**
- [ ] Después del pago, recibo con: ID tx, comercio, monto, firma digital
- [ ] Opción de descargar o compartir
- [ ] Recibo disponible en historial

---

**HU-023 [P1]** Como cliente, quiero transferir saldo a otro cliente ViCheck sin POS intermediario, para pagar a amigos o familia.

**Criterios de aceptación:**
- [ ] Botón "Pagar a persona"
- [ ] Input: teléfono o UID del destinatario
- [ ] Mismo flujo de pago NFC/PIN/confirmación

---

**HU-024 [P0]** Como cliente, quiero que el sistema me avise si no tengo saldo suficiente antes de intentar.

**Criterios de aceptación:**
- [ ] Antes de la firma, el POS verifica balance local
- [ ] Si es insuficiente, muestra "Saldo insuficiente" con monto faltante

---

## US-C-004 — Historial

**HU-030 [P0]** Como cliente, quiero ver mis últimas transacciones, para saber en qué gasté.

**Criterios de aceptación:**
- [ ] Home muestra últimas 10
- [ ] Pantalla de historial muestra paginación
- [ ] Cada tx muestra: fecha, tipo, monto, comercio (si aplica), estado

---

**HU-031 [P1]** Como cliente, quiero filtrar mis transacciones por fecha, tipo o monto, para encontrar una específica.

**Criterios de aceptación:**
- [ ] Filtros: rango de fechas, tipo (cashin/pay/cashout), monto min/max
- [ ] Resultados actualizados en vivo

---

**HU-032 [P1]** Como cliente, quiero exportar mi historial, para tener un registro personal.

**Criterios de aceptación:**
- [ ] Botón "Exportar" genera CSV
- [ ] Compartir vía email/drive/whatsapp

---

## US-C-005 — Seguridad

**HU-040 [P0]** Como cliente, quiero cambiar mi PIN, para mantener mi cuenta segura.

**Criterios de aceptación:**
- [ ] Settings → Seguridad → Cambiar PIN
- [ ] Validar PIN actual
- [ ] Nuevo PIN con confirmación

---

**HU-041 [P0]** Como cliente, quiero emparejar mi pulsera NFC con mi billetera, para pagar sin sacar el celular.

**Criterios de aceptación:**
- [ ] Settings → NFC → "Emparejar nuevo tag"
- [ ] Pantalla "Acerca el tag"
- [ ] Cuando lo detecta, pide PIN de confirmación
- [ ] Tag queda asociado a la wallet

---

**HU-042 [P0]** Como cliente, quiero cerrar mi cuenta, para ejercer mi derecho al olvido.

**Criterios de aceptación:**
- [ ] Settings → Cerrar cuenta
- [ ] Confirmación con PIN
- [ ] Confirmación de balance pendiente (se liquida antes de cerrar)
- [ ] Borrado de datos personales en 30 días

---

**HU-043 [P1]** Como cliente, quiero ver un historial de sesiones (dónde y cuándo se usó mi cuenta), para detectar accesos sospechosos.

**Criterios de aceptación:**
- [ ] Settings → Seguridad → Sesiones activas
- [ ] Lista con dispositivo, ubicación, último acceso
- [ ] Opción de cerrar sesión en otros dispositivos

---

## US-C-006 — Offline

**HU-050 [P0]** Como cliente, quiero ver claramente si estoy online u offline, para saber qué funciones están disponibles.

**Criterios de aceptación:**
- [ ] Indicador persistente en top bar: verde (online) / gris (offline)
- [ ] Mensaje contextual cuando se intenta CashIn offline: "Necesitas conexión para recargar"

---

**HU-051 [P0]** Como cliente, quiero que mis pagos offline se sincronicen automáticamente cuando vuelva internet, para que mi balance esté al día.

**Criterios de aceptación:**
- [ ] Indicador de "N transacciones pendientes de sync"
- [ ] Cuando vuelve internet, animación de sync
- [ ] Confirmación de cuántas tx se sincronizaron

---

## Resumen

| Historia | P0 | P1 |
|---|---|---|
| Onboarding | 3 | 0 |
| CashIn | 2 | 1 |
| Pago | 4 | 1 |
| Historial | 1 | 2 |
| Seguridad | 3 | 1 |
| Offline | 2 | 0 |
| **Total cliente** | **15** | **5** |

---

**Próximo documento:** `US-prestador.md` — historias del comercio.