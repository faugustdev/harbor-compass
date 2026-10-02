# Historias de Usuario: Prestador (Comercio)

> Prestador = comercio afiliado que usa el POS app ViCheck para cobrar a sus clientes.

---

## US-P-001 — Onboarding comercio

**HU-100 [P0]** Como nuevo prestador, quiero registrarme como comercio en ViCheck, para empezar a cobrar con la app POS.

**Criterios de aceptación:**
- [ ] Formulario: RIF, razón social, dirección, contacto
- [ ] Datos bancarios (banco + número de cuenta)
- [ ] Validación con VIPPO (RIF válido, banco válido)
- [ ] Tenant creado en estado `active`

---

**HU-101 [P0]** Como prestador, quiero configurar mi branding (logo, color), para personalizar la experiencia de mis clientes.

**Criterios de aceptación:**
- [ ] Settings → Branding
- [ ] Subir logo (PNG/SVG, ≤ 2MB)
- [ ] Elegir color primario (paleta)
- [ ] Preview de cómo se ve en la app del cliente

---

**HU-102 [P1]** Como prestador, quiero configurar mi fee por transacción, para saber cuánto se me cobra por usar ViCheck.

**Criterios de aceptación:**
- [ ] Settings → Fees
- [ ] Ver fee actual configurado por VIPPO
- [ ] (Admin) Cambiar fee por excepción

---

## US-P-002 — POS Cobro

**HU-110 [P0]** Como prestador, quiero cobrar a un cliente acercando su tag NFC al POS, para hacer la venta rápido.

**Criterios de aceptación:**
- [ ] Pantalla POS principal con monto
- [ ] Botón "Cobrar con NFC"
- [ ] Pantalla "Acerca el tag del cliente"
- [ ] Animación cuando detecta tag
- [ ] Validación del cliente (existe, que tenga saldo)
- [ ] Confirmación con cliente (PIN si monto ≥ umbral)
- [ ] Notificación de éxito

---

**HU-111 [P0]** Como prestador, quiero cobrar a un cliente escaneando mi QR desde su celular, para cobrar sin NFC.

**Criterios de aceptación:**
- [ ] POS genera QR dinámico (dChip) cada 30s
- [ ] Cliente escanea el QR desde su app
- [ ] Cliente autoriza con PIN
- [ ] POS confirma venta

---

**HU-112 [P0]** Como prestador, quiero cobrar un monto manual cuando el cliente no tiene tag ni celular, para no perder ventas.

**Criterios de aceptación:**
- [ ] Botón "Cobro manual"
- [ ] Input: teléfono del cliente, monto
- [ ] Cliente recibe push para aceptar
- [ ] Requiere confirmación del cliente

---

**HU-113 [P0]** Como prestador, quiero cancelar un cobro en proceso si me equivoco, para corregir errores.

**Criterios de aceptación:**
- [ ] Botón "Cancelar" visible durante el flujo de cobro
- [ ] Si tx no se firmó, se cancela sin efecto
- [ ] Si tx se firmó pero no se almacenó, se crea tx de cancelación

---

**HU-114 [P0]** Como prestador, quiero ver el estado de la transacción después del cobro, para confirmar que se procesó.

**Criterios de aceptación:**
- [ ] Pantalla post-venta con: cliente UID, monto, hora, firma
- [ ] Botón "Nueva venta" para empezar otra
- [ ] Botón "Recibo" para generar PDF

---

**HU-115 [P1]** Como prestador, quiero cobrar múltiples productos en una sola transacción, para ventas con varios items.

**Criterios de aceptación:**
- [ ] Carrito de productos
- [ ] Suma de precios
- [ ] Una sola transacción con el total

---

## US-P-003 — Sync

**HU-120 [P0]** Como prestador, quiero ver cuántas transacciones tengo sin sincronizar, para saber si hay pendientes.

**Criterios de aceptación:**
- [ ] Badge en POS con número de tx pendientes
- [ ] Tooltip: "X transacciones se subirán cuando vuelva internet"

---

**HU-121 [P0]** Como prestador, quiero que las transacciones se sincronicen automáticamente cuando vuelva internet, para no tener que hacerlo yo.

**Criterios de aceptación:**
- [ ] Auto-sync en background
- [ ] Indicador de progreso
- [ ] Notificación push al completar

---

**HU-122 [P1]** Como prestador, quiero forzar sync manualmente, para asegurar que las tx se procesen ya.

**Criterios de aceptación:**
- [ ] Botón "Sincronizar ahora"
- [ ] Útil si hay problemas de red

---

## US-P-004 — Reportes y Dashboard

**HU-130 [P0]** Como prestador, quiero ver las ventas del día, para saber cuánto llevo.

**Criterios de aceptación:**
- [ ] POS home muestra: total vendido hoy, # de transacciones
- [ ] Click lleva al detalle

---

**HU-131 [P0]** Como prestador, quiero ver ventas por rango de fechas, para análisis de negocio.

**Criterios de aceptación:**
- [ ] Selector de rango (hoy, semana, mes, custom)
- [ ] Total vendido, # tx, ticket promedio
- [ ] Gráfico de barras por día

---

**HU-132 [P1]** Como prestador, quiero ver el detalle de cada transacción, para auditoría.

**Criterios de aceptación:**
- [ ] Lista paginada con filtros
- [ ] Detalle con: cliente UID, monto, hora, firma, estado sync

---

**HU-133 [P1]** Como prestador, quiero exportar mis ventas, para contabilidad.

**Criterios de aceptación:**
- [ ] Botón "Exportar"
- [ ] Formato: CSV / PDF
- [ ] Compatible con software contable

---

## US-P-005 — CashOut

**HU-140 [P0]** Como prestador, quiero liquidar mi saldo a mi cuenta bancaria, para tener el dinero en mi banco.

**Criterios de aceptación:**
- [ ] Pantalla CashOut
- [ ] Input: monto (o "todo")
- [ ] Confirmación con PIN
- [ ] Recibo con ID de liquidación

---

**HU-141 [P0]** Como prestador, quiero ver el historial de mis liquidaciones, para saber cuándo recibí dinero.

**Criterios de aceptación:**
- [ ] Lista con: fecha, monto, banco destino, estado
- [ ] Detalle de cada liquidación

---

**HU-142 [P1]** Como prestador, quiero configurar auto-liquidación cuando se acumule un monto, para no tener que acordarme.

**Criterios de aceptación:**
- [ ] Settings → Auto-liquidación
- [ ] Configurable: umbral, frecuencia (diario/semanal)
- [ ] El sistema ejecuta CashOut automático

---

## US-P-006 — Operación

**HU-150 [P0]** Como prestador, quiero cerrar sesión cuando termino mi turno, para que el siguiente empleado use su cuenta.

**Criterios de aceptación:**
- [ ] Botón "Cerrar sesión"
- [ ] Confirma con credencial
- [ ] Sesión del POS queda limpia

---

**HU-151 [P0]** Como prestador, quiero cambiar mi PIN, para mantener seguridad.

**Criterios de aceptación:**
- [ ] Settings → Cambiar PIN
- [ ] Validar PIN actual
- [ ] Nuevo PIN con confirmación

---

**HU-152 [P1]** Como prestador, quiero ver mi balance disponible vs balance pendiente de sync, para saber cuánto puedo liquidar.

**Criterios de aceptación:**
- [ ] Mostrar ambos balances separados
- [ ] Solo se puede liquidar el balance sincronizado

---

## US-P-007 — Soporte

**HU-160 [P0]** Como prestador, quiero reportar un problema técnico, para que me ayuden.

**Criterios de aceptación:**
- [ ] Pantalla "Reportar problema"
- [ ] Categoría: bug, transacción, sync, otro
- [ ] Input de descripción
- [ ] Adjuntar screenshot (opcional)
- [ ] Envío vía backend ViCheck

---

**HU-161 [P1]** Como prestador, quiero ver el estado de mis tickets de soporte, para saber si están resueltos.

**Criterios de aceptación:**
- [ ] Lista de tickets
- [ ] Estado: abierto / en proceso / resuelto
- [ ] Comunicación con soporte

---

## Resumen

| Historia | P0 | P1 |
|---|---|---|
| Onboarding | 2 | 1 |
| POS cobro | 4 | 1 |
| Sync | 2 | 1 |
| Reportes | 2 | 2 |
| CashOut | 2 | 1 |
| Operación | 2 | 1 |
| Soporte | 1 | 1 |
| **Total prestador** | **15** | **8** |

---

**Próximo documento:** `US-operador-transporte.md` — historias del operador de transporte.