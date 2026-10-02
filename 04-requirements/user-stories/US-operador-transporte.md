# Historias de Usuario: Operador de Transporte

> Operador = empresa de transporte (buses, taxis) que usa ViCheck como sistema de ticketing.
> Caso piloto: Terminal La Bandera y rutas interurbanas.

---

## US-T-001 — Onboarding operador

**HU-200 [P0]** Como operador de transporte, quiero registrar mi flota de unidades, para que cada vehículo tenga su POS vinculado.

**Criterios de aceptación:**
- [ ] Formulario: # de placa, modelo, año, ruta
- [ ] Vinculación con device_id del POS (tablet/celular del vehículo)
- [ ] Cada unidad tiene tenant_id independiente o comparte con operador

---

**HU-201 [P0]** Como operador, quiero configurar tarifas por ruta, para cobrar el pasaje correcto.

**Criterios de aceptación:**
- [ ] Lista de rutas
- [ ] Cada ruta tiene tarifa fija
- [ ] Opción de tarifa por tramos

---

**HU-202 [P1]** Como operador, quiero configurar tarifas por horario (pico/no pico), para optimizar ingresos.

**Criterios de aceptación:**
- [ ] Tarifas diferenciadas por horario
- [ ] Calendarización (lun-vie, fines de semana)

---

## US-T-002 — Operación en ruta

**HU-210 [P0]** Como conductor, quiero cobrar el pasaje a cada pasajero al abordar, para registrar el ingreso.

**Criterios de aceptación:**
- [ ] Pantalla POS muestra tarifa fija (o permite seleccionar tramos)
- [ ] Pasajero acerca su tag NFC
- [ ] Confirmación con PIN del pasajero (si monto ≥ umbral)
- [ ] Tx registrada
- [ ] Siguiente pasajero

---

**HU-211 [P0]** Como conductor, quiero ver cuántas personas han abordado en mi unidad, para control de ruta.

**Criterios de aceptación:**
- [ ] Contador de pasajeros hoy
- [ ] Total recaudado hoy
- [ ] Sincronización pendiente

---

**HU-212 [P0]** Como conductor, quiero cobrar abonos mensuales a pasajeros frecuentes, para no cobrar cada vez.

**Criterios de aceptación:**
- [ ] Detección de tag con abono activo
- [ ] Cobra 1 viaje del abono
- [ ] Cuando se acaba el abono, se le avisa al pasajero

---

**HU-213 [P1]** Como conductor, quiero imprimir boletos para pasajeros que no tienen ViCheck, para no perder ventas.

**Criterios de aceptación:**
- [ ] Botón "Venta de boleto" sin NFC
- [ ] Imprime en mini-printer Bluetooth del vehículo
- [ ] Registra venta en backend (efectivo)

---

## US-T-003 — Reportes

**HU-220 [P0]** Como operador, quiero ver ventas por unidad y ruta, para control de flota.

**Criterios de aceptación:**
- [ ] Dashboard con lista de unidades
- [ ] Cada unidad: ventas hoy, # pasajeros, promedio
- [ ] Drill-down por unidad específica

---

**HU-221 [P0]** Como operador, quiero ver ventas consolidadas por ruta, para análisis de demanda.

**Criterios de aceptación:**
- [ ] Reporte por ruta
- [ ] Total recaudado, # pasajeros, promedio por horario

---

**HU-222 [P0]** Como operador, quiero liquidar las ventas del día al banco, para tener control financiero.

**Criterios de aceptación:**
- [ ] CashOut al banco de la empresa
- [ ] Frecuencia configurable (diario, semanal)
- [ ] Reporte fiscal compatible

---

**HU-223 [P1]** Como operador, quiero comparar ventas entre unidades, para detectar anomalías.

**Criterios de aceptación:**
- [ ] Reporte comparativo
- [ ] Alertas si una unidad tiene desviación estándar

---

## US-T-004 — Anti-fraude

**HU-230 [P0]** Como operador, quiero detectar tags clonados o fraudulentos, para evitar pérdidas.

**Criterios de aceptación:**
- [ ] Backend valida firma ECDSA de cada tag
- [ ] Tags sin firma válida son rechazados
- [ ] Log de intentos fraudulentos

---

**HU-231 [P0]** Como operador, quiero detectar "robo hormiga" (cobros no registrados), para que no se queden con el efectivo.

**Criterios de aceptación:**
- [ ] Cada tx queda registrada en backend (aunque offline)
- [ ] Reconciliación detecta discrepancias
- [ ] Alerta si # de pasajeros vs # tx no cuadra

---

**HU-232 [P1]** Como operador, quiero limitar la frecuencia de uso de un mismo tag, para evitar fraude.

**Criterios de aceptación:**
- [ ] Backend detecta uso atípico
- [ ] Configurable: máx N tx/hora por UID

---

## US-T-005 — Operación

**HU-240 [P0]** Como conductor, quiero iniciar y cerrar turno, para delimitar mi responsabilidad.

**Criterios de aceptación:**
- [ ] Botón "Iniciar turno" al comenzar
- [ ] Botón "Cerrar turno" al finalizar
- [ ] Resumen de ventas del turno

---

**HU-241 [P0]** Como conductor, quiero reportar problemas técnicos en ruta, para que me ayuden.

**Criterios de aceptación:**
- [ ] Pantalla de reporte en POS
- [ ] Categorías: NFC, sync, dinero, otro
- [ ] Envío al backend

---

## US-T-006 — Piloto La Bandera

**HU-250 [P0]** Como operador del Terminal La Bandera, quiero integrar ViCheck con el sistema de conteo actual, para no duplicar sistemas.

**Criterios de aceptación:**
- [ ] Exportar ventas ViCheck al sistema de conteo
- [ ] O: importar conteo de otros sistemas a ViCheck
- **Status:** integración a definir con el operador

---

**HU-251 [P1]** Como operador, quiero mostrar el saldo del pasajero antes de cobrar, para confirmar transacción.

**Criterios de aceptación:**
- [ ] POS muestra balance del cliente antes de cobrar
- [ ] Mensaje si saldo < tarifa

---

## Resumen

| Historia | P0 | P1 |
|---|---|---|
| Onboarding | 1 | 1 |
| Operación ruta | 3 | 1 |
| Reportes | 3 | 1 |
| Anti-fraude | 2 | 1 |
| Operación | 2 | 0 |
| Piloto | 1 | 1 |
| **Total operador** | **12** | **5** |

---

**Próximo documento:** `US-admin.md` — historias del admin VIPPO.