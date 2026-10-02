# ViCheck MVP — Product Brief

## 1. Problema

Venezuela tiene la **infraestructura de telecomunicaciones más inestable de la región**: cortes de electricidad, zonas sin cobertura 4G, intermitencia de WiFi en zonas comerciales. Esto bloquea el flujo de pago digital en contextos donde el efectivo es inseguro o impracticable:

- **Transporte público** (terminales como La Bandera, rutas interurbanas): el pasajero sube con efectivo, el conductor maneja miles de bolívares sin registro, hay "robo hormiga" crónico, los reportes al final del día son manuales y mentirosos
- **Pequeño (comida rápida, ferias, eventos masivos, vendedores ambulantes):** el comercio pierde ventas cuando la red falla
- **Pago entre personas (P2P):** en zonas rurales o apagones, no se puede transferir

VIPPO ya tiene **SmartCashier** (POS digital) y **billetera electrónica con 65K+ usuarios**. ViCheck añade la capa de **supervivencia offline** que falta.

## 2. Propuesta de valor

**ViCheck = billetera offline-first** que:

- Permite **pagar incluso sin internet** (chip-to-phone o phone-to-phone)
- **Liquida automáticamente** cuando vuelve la conexión
- **Elimina el manejo de efectivo** en contextos donde el cash es inseguro o lento
- **Registra todas las transacciones** sin depender de la red para validación
- **Auditable** por el comercio y por el regulador (SENIAT, SUDEBAN)

## 3. Diferenciador vs MycCashless

| Capacidad | MycCashless | ViCheck (propuesto) |
|---|---|---|
| Offline transaccional | ✅ | ✅ |
| Phone-to-phone (P2P) | ✅ (sync engine) | ✅ |
| Chip-to-phone (NFC/UID) | ✅ | ✅ |
| QR dinámico autodegradable | ✅ (dChip) | ✅ |
| Auto-auditoría | ✅ | ✅ |
| **Integración con banco local VE** | ❌ (Stripe only) | ✅ (vía APIs VIPPO existentes) |
| **Multi-comercio: pago pequeño** | ❌ (orientado a eventos) | ✅ (transporte, comida, ferias) |
| **Multi-tenancy** | ✅ | ✅ |
| **Despliegue en zona con red inestable** | ⚠️ (WiFi requerido en fin de lugar) | ✅ (sin red) |

## 4. Usuarios objetivo

### 4.1 Cliente final (pasajero / consumidor)
Persona que carga saldo en ViCheck y paga con su celular o pulsera NFC en cualquier comercio afiliado. Quiere velocidad y no depender del estado de la conexión.

### 4.2 Prestador de servicios (comercio afiliado)
Negocio que cobra a sus clientes usando la app POS de ViCheck. Necesita: registrar ventas, conciliar, recibir liquidación en su cuenta bancaria. Quiere: cero pérdida de ventas por fallas de conectividad.

### 4.3 Operador de transporte
Empresa de transporte (buses, taxis, mototaxis) que usa ViCheck como sistema de ticketing. Caso de uso piloto en **Terminal La Bandera** y rutas interurbanas.

### 4.4 Admin VIPPO
Interno: onboarding de comercios, monitoreo de transacciones, anti-fraude, conciliaciones masivas, soporte.

## 5. Casos de uso prioritarios (MVP)

| # | Caso | Prioridad | Piloto |
|---|---|---|---|
| UC-01 | CashIn online | P0 | ✅ |
| UC-02 | Pago offline phone-to-phone | P0 | ✅ |
| UC-03 | Pago offline chip-to-phone (NFC) | P0 | ✅ |
| UC-04 | Pago offline QR dinámico (dChip) | P0 | ✅ |
| UC-05 | Sync + CashOut online | P0 | ✅ |
| UC-06 | Conciliación bancaria | P1 | ✅ |
| UC-07 | Multi-tenancy (múltiples comercios) | P1 | ✅ |
| UC-08 | Reportes / dashboard comercio | P2 | ⏸️ post-MVP |
| UC-09 | Anti-fraude (anomaly monitoring) | P2 | ⏸️ post-MVP |
| UC-10 | dChip web fallback (sin app instalada) | P1 | ⏸️ post-MVP |

## 6. Out of scope para MVP
- iOS (solo Android por restricción de recursos)
- Anti-fraude con ML (solo reglas básicas)
- Onboarding biométrico / KYC (delegado a VIPPO)
- Marketplace de productos / catálogo
- Webhooks a sistemas terceros
- Multi-moneda (solo VESB y VEBITDA/USD)
- SDK abierto para terceros

## 7. Métricas de éxito (KPIs) del MVP

| Métrica | Target MVP | Cómo se mide |
|---|---|---|
| Latencia de transacción offline (tap → confirmación) | < 300 ms | Métrica en POS |
| Transacciones por segundo (TPS) por nodo POS | ≥ 5 TPS | Load test |
| Sync time cuando vuelve la red (1000 tx pendientes) | < 30 segundos | Métrica backend |
| Tiempo CashIn (online) | < 5 segundos | Métrica backend |
| Tasa de éxito de CashIn | ≥ 95% | Logs |
| Tasa de éxito de transacciones offline | ≥ 99% | Logs |
| Tiempo medio de CashOut al banco | < 24 horas | Logs bancarios |
| Multi-tenancy simultáneo | 10 comercios independientes | Test funcional |
| Crash rate Android | < 1% | Crashlytics |

## 8. Restricciones / Asunción

| # | Restricción | Impacto |
|---|---|---|
| R-1 | Sin internet garantizado en Venezuela | Diseño offline-first obligatorio |
| R-2 | Recursos limitados del equipo | Solo Android, sin iOS en MVP |
| R-3 | Multi-tenant desde día 1 | Arquitectura debe separar datos por tenant |
| R-4 | Integración con APIs VIPPO existentes | CashIn/CashOut deben consumir las APIs existentes |
| R-5 | Despliegue en Google Play | Cumplir políticas de Play Store |
| R-6 | Datos sensibles financieros | Cumplimiento y encriptación de criptografía fuerte |
| R-7 | Costo infraestructura bajo | Serverless AWS SAM + PostgreSQL gestionado |
| R-8 | Multi-tenancy: cada comercio es tenant | Aislamiento de saldos, transacciones y reportes |

## 9. Stakeholders

- **Product Owner:** Francisco August (consultor externo VIPPO)
- **Sponsor VIPPO:** Miguel León (fundador VIPPO)
- **Equipo técnico VIPPO:** backend legacy (SmartCashier, FIS-CORTEX), APIs de pago
- **Usuarios piloto:** operador de transporte La Bandera, primer comercio Chino
- **Audiencia beta:** cerrada, < 50 usuarios iniciales

## 10. Definición de "MVP completo"

El MVP está completo cuando:
1. Cliente puede hacer CashIn desde su banco VIPPO
2. Cliente puede pagar a un comercio sin internet (NFC o dChip QR)
3. Cliente puede pagar P2P (phone-to-phone) sin internet
4. POS comercio registra transacción en bóveda local encriptada
5. Cuando vuelve internet, sync automático de transacciones pendientes
6. Comercio puede ver reporte de ventas y solicitar CashIn
8. 10 comercios pueden operar simultáneamente con sus salves y sep
9. App en Google Play cerrada (internal test track)

---

**Próximo documento:** `02-architecture.md` — la arquitectura técnica del motor OffVi.