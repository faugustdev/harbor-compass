# ViCheck MVP — Estimación de Tiempo

## Premisas

- **Recursos:** 1 dev full-stack (Francisco, consultor externo VIPPO)
- **Stack:** Kotlin Multiplatform (Android) + Python AWS SAM (backend)
- **Alcance MVP:** 32 requisitos P0 + 11 P1 parciales
- **No incluye:** app iOS, dashboard admin, anti-fraude con ML

## Estimación por fase (semanas)

```
Fase 0 — Setup & arquitectura            ████░░░░░░░░░░░░  2 sem
Fase 1 — Backend core (auth, sync, ledger) █████░░░░░░░░░░░  4 sem
Fase 2 — Android core (wallet, NFC)       ██████░░░░░░░░░░  5 sem
Fase 3 — Integración VIPPO APIs           ███░░░░░░░░░░░░░  3 sem
Fase 4 — Piloto + testing + ajustes       ████░░░░░░░░░░░░  4 sem
Fase 5 — Play Store + cierre MVP          ██░░░░░░░░░░░░░░  2 sem
                                          ─────────────────────
                                          TOTAL: 20 semanas ≈ 5 meses
```

## Desglose por fase

### Fase 0: Setup & arquitectura (semana 1-2)
- Configurar monorepo con KMP
- Setup AWS SAM + Lambda
- Diseño de DB schema (este doc)
- Setup CI/CD básico
- Setup monitoring (CloudWatch + Crashlytics)
- **Entregable:** Repo + infraestructura funcional

### Fase 1: Backend core (semana 3-6)
- Lambda endpoints: auth, sync, balance, cashin, cashout
- PostgreSQL schema + migrations
- Redis cache
- Tests unitarios + integration
- **Entregable:** Backend funcional con tests

### Fase 2: Android core (semana 5-9)
- KMP shared core (modelos, crypto, sync)
- UI Android con Compose (Home, Wallet, Pay, CashIn)
- NFC module (Foreground Dispatch)
- QR module (dChip)
- Vault local encriptado (Room + SQLCipher)
- Tests unitarios
- **Entregable:** App Android funcional con todos los flujos core

### Fase 3: Integración VIPPO APIs (semana 9-11)
- Cliente HTTP a VIPPO `/validacion de pago`
- Cliente HTTP a VIPPO `/cuentas`
- Cliente HTTP a VIPPO `/liquidacion`
- Webhook receiver
- **Entregable:** End-to-end con datos reales de VIPPO

### Fase 4: Piloto + testing (semana 11-14)
- Beta cerrada con 5-10 usuarios reales
- Load testing backend
- Crash testing app
- Ajustes UX + bugs
- **Entregable:** App validada en campo

### Fase 5: Play Store (semana 14-15)
- Privacy policy + data safety form
- Internal testing track
- Screenshots, descriptions
- Fastlane setup
- **Entregable:** App en Play Store (closed beta)

## Buffer

- **+2 semanas buffer** entre fase 4 y 5 por hallazgos
- **Total realista: 5.5 - 6 meses** desde kickoff

## Hitos clave

| Hito | Fecha | Criterio |
|---|---|---|
| Backend deployable | semana 6 | API + DB + tests |
| App compilable | semana 9 | UI + flows |
| E2E funcional | semana 11 | CashIn + Pay con VIPPO |
| Beta cerrada | semana 14 | 5+ usuarios probando |
| Play Store beta | semana 15-16 | App en internal testing |

## Riesgos que pueden extender el timeline

| Riesgo | Impacto | Mitigación |
|---|---|---|
| APIs VIPPO no documentadas | +1-2 semanas | Conseguir acceso a docs temprano |
| Latencia de pruebas en beta | +1 semana | Empezar beta con usuarios internos |
| Cambios regulatorios VE | +1-2 semanas | Auditoría legal temprana |
| KMP issues | +1 semana | Empezar con PoC de KMP semana 1 |
| Multi-tenancy complejo | +1 semana | RLS policies + auditorías semana 2 |

## Recursos adicionales que podrían acelerar

- **1 backend dev adicional:** reduce Fase 1-3 en 30%
- **Diseñador UX:** reduce Fase 2-4 en 25%
- **QA tester:** reduce Fase 4 en 50%

## Lo que NO está en esta estimación

- ❌ App iOS (post-MVP)
- ❌ Dashboard admin web (post-MVP)
- ❌ Anti-fraude ML (post-MVP)
- ❌ SDK abierto (post-MVP)
- ❌ Multi-moneda (post-MVP)
- ❌ Marketing / lanzamiento
- ❌ Soporte operacional continuo

---

**Próximo documento:** `08-questions-for-vippo.md`