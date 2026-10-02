# ViCheck MVP — Billetera Offline-First

Billetera móvil con **Motor de Sincronización Asíncrono (OffVi)** para pagos **pequeño a pequeño (P2P) y comercio a cliente** que sobrevive a la pérdida de conectividad.

**Estado:** MVP en diseño. Closed beta.
**Owner:** Francisco August (consultor externo VIPPO)
**Cliente:** VIPPO C.A. (Caracas, Venezuela)
**Última actualización:** 2026-10-02

---

## Tabla de contenidos

```
docs/
├── README.md                       ← este archivo (índice)
├── 01-product-brief.md            ← brief del producto, problema, propuesta de valor
├── 02-architecture.md             ← arquitectura técnica (OffVi Sync Engine)
├── 03-tech-stack.md               ← stack propuesto + justificación
├── 04-requirements/
│   ├── functional-requirements.md ← requisitos funcionales (RF-xxx)
│   ├── non-functional.md          ← requisitos no funcionales (seguridad, performance, etc.)
│   ├── user-stories/
│   │   ├── US-cliente-final.md
│   │   ├── US-prestador.md
│   │   ├── US-operador-transporte.md
│   │   └── US-admin.md
│   └── use-cases/
│       ├── UC-01-cashin.md
│       ├── UC-02-pago-offline.md
│       ├── UC-03-pago-nfc.md
│       ├── UC-04-pago-qr-dchip.md
│       ├── UC-05-sync-cashout.md
│       └── UC-06-conciliacion.md
├── 05-data-model.md               ← modelo de datos (PostgreSQL + esquema sync)
├── 06-api-contract.md             ← contrato de APIs REST + WebSocket
├── 07-time-estimate.md            ← estimación de tiempo por fase / entregable
├── 08-questions-for-vippo.md      ← preguntas técnicas para reunión de alineación
└── 09-decisions/
    └── ADR-001-multiplatform.md   ← Architecture Decision Records
```

---

## Quick start para el lector

1. **Producto y problema:** `01-product-brief.md`
2. **Arquitectura propuesta:** `02-architecture.md`
3. **Stack y justificación:** `03-tech-stack.md`
4. **Requisitos funcionales:** `04-requirements/functional-requirements.md`
5. **Historias de usuario:** `04-requirements/user-stories/`
6. **Casos de uso (flujos):** `04-requirements/use-cases/`
7. **Modelo de datos:** `05-data-model.md`
8. **API contract:** `06-api-contract.md`
9. **Estimación de tiempo:** `07-time-estimate.md`
10. **Preguntas abiertas para VIPPO:** `08-questions-for-vippo.md`

---

## Glosario mínimo

- **CashIn:** carga de saldo online, desde banco del cliente hacia ViCheck
- **CashOut:** liquidación de saldo acumulado hacia cuenta bancaria del prestador
- **OffVi:** motor de sincronización asíncrono offline (Offline-VIPPO)
- **dChip:** QR dinámico autogenerado, autodegradable, no replicable por screenshot
- **Bóveda Encriptada:** SQLite local encriptado (AES-256) que guarda transacciones pendientes
- **Void:** estado sin conectividad donde ocurren las transacciones offline
- **Conciliación:** proceso de cuadrar transacciones offline con el ledger central cuando vuelve la red
- **Identificador físico (PhyID):** UID único de chip NFC (pulsera, tarjeta, celular)
- **Identificador lógico (LogID):** dChip QR dinámico equivalente a un PhyID

---

## Documentos relacionados en el monorepo

- `../backend/redesigned-couscous/` — stack AWS SAM + Python3.14 de referencia del proyectos
- `../backend/template.yaml` — template base de Serverless Application Model

---

## Convenciones del repositorio

- Markdown para todo documento
- Diagramas en Mermaid (renderiza GitHub/GitLab)
- Conventional commits YAML
- ADR (Architecture Decision Records) en `09-decisions/`
- Historias de usuario formato: "Como [rol], quiero [acción], para [beneficio]"