# ViCheck MVP — Product Brief (Modelo corregido)

## 1. Problema

Venezuela tiene la **infraestructura de telecomunicaciones más inestable de la región**: cortes de electricidad, zonas sin cobertura 4G, intermitencia de WiFi en zonas comerciales. Esto bloquea el flujo de pago digital en contextos donde el efectivo es inseguro o impracticable.

VIPPO ya tiene SmartCashier (POS digital) y billetera electrónica. ViCheck añade la capa de **supervivencia offline** que falta, manteniendo el modelo donde **ViCheck nunca custodia dinero**.

## 2. Propuesta de valor

**ViCheck = capa de orquestación de pagos offline-first** que:

- Permite al cliente **pagar a un aliado** que ya recibió el dinero en su cuenta bancaria
- **Sincroniza** automáticamente cuando vuelve la conexión
- **Funciona offline** una vez que el cliente cargó su wallet (CashIn)
- **Elimina el manejo de efectivo** en contextos donde el cash es inseguro
- **Registra todas las transacciones** sin depender de la red para validación
- **Permite transferencia entre aliados** (cliente pasa saldo de un aliado a otro)

## 3. Modelo de dinero — CORREGIDO

### Principio clave: NO somos custodio

```
┌────────────────────────────────────────────────────────────────┐
│  CLIENTE                                                     │
│  Su banco (origen)                                            │
│         │                                                    │
│         │ ① CashIn (online): autorización + registro         │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │ ViCheck       │  (registra, valida, NO toca dinero)       │
│  │ wallet lógica│                                            │
│  │ del cliente   │                                            │
│  └──────────────┘                                            │
│         │                                                    │
│         │ ② Dinero va directo a cuenta del Aliado            │
│         ▼                                                    │
│  ┌─────────────────────────────────────────┐                  │
│  │ ALIADO (su banco)                        │                  │
│  │ Recibe el dinero del CashIn              │                  │
│  └─────────────────────────────────────────┘                  │
│         ▲                                                    │
│         │ ③ Cliente paga con ViCheck                          │
│         │    (descuenta de su wallet lógica)                  │
│         │                                                    │
│  ┌──────────────┐                                            │
│  │ ViCheck       │  (sincroniza offline-first)                │
│  │ wallet lógica│                                            │
│  └──────────────┘                                            │
└────────────────────────────────────────────────────────────────┘
```

### Flujos de dinero

**CashIn (online):**
```
1. Cliente en ViCheck app selecciona "Recargar"
2. ViCheck valida con API VIPPO que el cliente tiene saldo en su banco
3. ViCheck registra autorización
4. Banco del cliente transfiere DIRECTO a cuenta del Aliado
5. ViCheck acredita saldo en la wallet lógica del cliente
   (el dinero YA está en la cuenta del Aliado, ViCheck solo registra)
```

**Pago online (con conexión):**
```
1. Cliente selecciona monto y aliado
2. ViCheck valida que el cliente tiene saldo en su wallet lógica
3. ViCheck descuenta de la wallet del cliente
4. Registra transacción (offline-first queue)
5. Confirma al cliente
```

**Pago offline (sin conexión):**
```
1. Cliente selecciona monto y aliado
2. ViCheck descuenta localmente de la wallet (signature ECDSA)
3. Registra en vault local encriptado
4. Confirma al cliente
5. Cuando vuelve internet → sync automático al backend
6. Backend concilia con la cuenta del Aliado
```

**Transferencia entre Aliados (online):**
```
1. Cliente tiene saldo acumulado con Aliado A
2. Quiere gastar en Aliado B
3. ViCheck valida:
   - Cliente tiene saldo suficiente con Aliado A
   - Aliado A tiene fondos suficientes en su banco
   - Restricciones aplicables (montos, frecuencia)
4. Banco del Aliado A transfiere a cuenta del Aliado B
5. ViCheck registra el movimiento
6. Cliente ahora tiene crédito con Aliado B
```

### Quién custodia qué

| Actor | Qué tiene/custodia |
|---|---|
| Cliente (persona) | Dinero en su cuenta bancaria propia |
| ViCheck | Nada. Solo es capa de orquestación y registro |
| Aliado (comercio) | Dinero en su cuenta bancaria propia |
| Banco del cliente | Fondos del cliente |
| Banco del aliado | Fondos del aliado |

**ViCheck NO custodia dinero en ningún momento.**

## 4. Diferenciador vs MycCashless

| Capacidad | MycCashless | ViCheck (corregido) |
|---|---|---|
| Offline transaccional | ✅ | ✅ |
| Phone-to-phone (P2P) | ✅ (sync engine) | ❌ (no es nuestro foco) |
| Chip-to-phone (NFC/UID) | ✅ | ✅ |
| QR dinámico autodegradable | ✅ (dChip) | ✅ |
| Custodia dinero del cliente | ✅ (evento cerrado) | ❌ (no custodia) |
| Transferencia entre aliados | ❌ | ✅ (interno, vía bancos) |
| **Integración con banco local VE** | ❌ (Stripe only) | ✅ (vía VIPPO APIs) |
| Multi-comercio: pago pequeño | ❌ (orientado a eventos) | ✅ (transporte, comida, ferias) |
| Multi-tenancy | ✅ | ✅ |
| Despliegue sin red | ⚠️ (WiFi requerido) | ✅ (sin red) |

## 5. Usuarios objetivo

### 5.1 Cliente final
Persona que carga saldo en su wallet ViCheck (contra su banco) y paga a aliados con NFC/QR. Quiere velocidad y no depender del estado de la conexión.

### 5.2 Prestador de servicios (Aliado/Comercio)
Negocio que cobra a sus clientes usando la app POS. Recibe el dinero directo en su cuenta bancaria. Necesita: registrar ventas, conciliar, ver dashboard.

### 5.3 Operador de transporte
Empresa de transporte (buses, taxis) que usa ViCheck. Caso piloto: Terminal La Bandera y rutas interurbanas.

### 5.4 Admin VIPPO
Interno: onboarding de aliados, monitoreo de transacciones, anti-fraude, conciliaciones, soporte.

## 6. Casos de uso prioritarios (MVP)

| # | Caso | Prioridad | Piloto |
|---|---|---|---|
| UC-01 | CashIn (online) | P0 | ✅ |
| UC-02 | Pago offline phone-to-phone | P0 | ✅ |
| UC-03 | Pago offline chip-to-phone (NFC) | P0 | ✅ |
| UC-04 | Pago offline QR dinámico (dChip) | P0 | ✅ |
| UC-05 | Sync + conciliación (online) | P0 | ✅ |
| UC-06 | Transferencia entre aliados | P1 | ⏸️ post-MVP |
| UC-07 | Multi-tenancy | P1 | ✅ |
| UC-08 | Reportes / dashboard aliado | P2 | ⏸️ post-MVP |
| UC-09 | Anti-fraude | P2 | ⏸️ post-MVP |

## 7. Restricciones críticas

### 7.1 Modo offline real
- Cliente carga wallet con CashIn (online)
- Una vez cargada, puede gastar offline
- Cada gasto offline se sincroniza cuando vuelve conexión
- Tope configurable por app Y backend

### 7.2 Pérdida de saldo por desinstalación
- Si cliente tiene dinero en wallet pero desinstala sin sincronizar → pierde el saldo
- Mecanismo de autorización explícito en la app
- "Al desinstalar sin sincronizar, aceptas perder X Bs"
- Sincronización forzada antes de permitir desinstalación (con countdown)

### 7.3 Motor de sincronización — corazón del producto
- Sincronización automática cuando vuelve internet
- Reintentos con backoff exponencial
- Cola persistente (sobrevive crashes)
- Validación contra backend al sincronizar
- Reconciliación entre vueltas

## 8. Compliance y regulatorio

### NO aplica
- **SUDEBAN** — no custodiamos dinero
- Licencia de entidad financiera
- Regulación de billetera electrónica

### SÍ aplica
- SENIAT — reportes fiscales
- LOPDP — protección de datos personales
- Regulaciones de interoperabilidad bancaria
- ISO 27001 — si manejamos datos sensibles

## 9. Out of scope para MVP
- iOS (solo Android)
- Anti-fraude con ML
- Onboarding biométrico / KYC (delegado a VIPPO)
- Marketplace
- Webhooks a sistemas terceros
- Multi-moneda (solo VES)
- SDK abierto
- Transferencia entre aliados (UC-06, post-MVP)

## 10. Métricas de éxito (KPIs) del MVP

| Métrica | Target MVP |
|---|---|
| Latencia transacción offline (tap → confirmación) | < 300 ms |
| Transacciones por segundo (TPS) por nodo POS | ≥ 5 TPS |
| Sync time (1000 tx pendientes) | < 30 s |
| CashIn response time | < 5 s |
| Tasa de éxito de CashIn | ≥ 95% |
| Tasa de éxito de transacciones offline | ≥ 99% |
| Crash rate Android | < 1% |
| 0% pérdida de saldo por desinstalación | sin sync confirmado |

## 11. Stakeholders

- Product Owner: Francisco August (consultor externo VIPPO)
- Sponsor VIPPO: Miguel León (fundador VIPPO)
- Equipo técnico VIPPO: backend legacy, APIs de pago
- Usuarios piloto: operador de transporte La Bandera, primer aliado

## 12. Definición de "MVP completo"

El MVP está completo cuando:
1. Cliente puede hacer CashIn desde su banco (vía VIPPO APIs)
2. Dinero va directo a cuenta del Aliado
3. Cliente puede gastar offline con NFC/QR después del CashIn
4. Cliente puede gastar online también
5. POS aliado registra transacción en bóveda local encriptada
6. Cuando vuelve internet, sync automático de transacciones pendientes
7. Aliado puede ver reporte de ventas (no saldo, solo ventas)
8. App en Google Play cerrada (internal test track)
9. Sincronización forzada antes de desinstalación funciona

---

**Próximo documento:** `02-architecture.md` (corregido)