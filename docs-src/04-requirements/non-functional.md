# ViCheck MVP — Requisitos No Funcionales (RNF)

## 1. Seguridad

### RNF-SEC-001 [P0] Encriptación en tránsito
Todas las comunicaciones cliente-servidor y servidor-servidor usan TLS 1.3.
- **Métrica:** 100% de requests sobre TLS
- **Validación:** penetration test

### RNF-SEC-002 [P0] Encriptación en reposo
La base de datos SQLite local del dispositivo está encriptada con AES-256.
- **Implementación:** SQLCipher con key derivada de Android Keystore

### RNF-SEC-003 [P0] Almacenamiento seguro de llaves
Llaves privadas ECDSA se almacenan en Android Keystore (hardware-backed cuando disponible).
- **Fallback:** EncryptedSharedPreferences con key derivada de master password

### RNF-SEC-004 [P0] Autenticación mutua
Cliente y servidor se autentican mutuamente con certificados / JWT firmado.
- **Cliente:** Bearer JWT con rotación cada 15min
- **Servidor:** API key para llamadas internas + OAuth para VIPPO APIs

### RNF-SEC-005 [P0] Protección contra replay attacks
Cada transacción incluye nonce único (UUID v4) + timestamp.
- **Backend:** rechaza tx con timestamp >5 min o nonce duplicado

### RNF-SEC-006 [P0] Rate limiting
Por wallet: máx 20 tx/min, 200 tx/h, 1000 tx/día
Por tenant: máx configurable
Por IP: máx 100 req/s
- **Implementación:** Redis con sliding window

### RNF-SEC-007 [P0] Auditoría
Toda operación sensible genera log de auditoría con timestamp, actor, acción, tenant.
- **Storage:** tabla `audit_log` append-only
- **Retención:** 7 años (regulatorio VE)

### RNF-SEC-008 [P1] Penetration testing
Antes de salir a producción, third-party pentest.
- **Scope:** APIs, app Android, vault local

### RNF-SEC-009 [P1] Bug bounty
Programa abierto para investigadores.
- **Budget MVP:** no en MVP, post-MVP

## 2. Performance

### RNF-PERF-001 [P0] Latencia NFC tap → confirmación
**≤ 300 ms desde tap a confirmación visual.**
- Latencia: < 50ms (NFC read)
- Latencia: < 100ms (local lookup + crypto)
- Latencia: < 100ms (UI render)

### RNF-PERF-002 [P0] Sync de 1000 tx pendientes
**≤ 30 segundos para subir batch completo.**
- Tamaño batch: 100 tx por HTTP request
- Requests paralelos: hasta 10
- Throughput esperado: 100 tx/s por cliente

### RNF-PERF-003 [P0] CashIn response time
**≤ 5 segundos para confirmar CashIn (online).**
- Latency end-to-end con VIPPO API incluida

### RNF-PERF-004 [P0] Backend TPS
**≥ 100 TPS sostenibles en Lambda.**
- Validación: load test con 1000 RPS, p95 < 1s

### RNF-PERF-005 [P0] Cold start Android
**≤ 2 segundos desde launch a primera pantalla.**
- Mitigación: lazy init, splash simple, WorkManager warm-up

### RNF-PERF-006 [P0] Tamaño APK
**< 50 MB APK**
- Sin recursos pesados embebidos
- Vector drawables en vez de bitmaps

### RNF-PERF-007 [P1] Battery impact
**< 5% battery por hora de uso activo.**
- NFC Foreground Dispatch optimizado
- Sync solo cuando hay tx pendientes
- WorkManager con constraints (WiFi, charging)

## 3. Disponibilidad

### RNF-AVAIL-001 [P0] Uptime backend
**≥ 99.5% mensual** (no es SLA estricto, es objetivo)
- **Mitigación:** Multi-AZ de RDS, API Gateway redundante

### RNF-AVAIL-002 [P0] Offline functionality
**100% de operaciones críticas funcionan sin internet:**
- Pago NFC
- Pago QR
- Pago P2P
- Ver historial local
- Ver balance local

### RNF-AVAIL-003 [P0] Auto-sync cuando vuelve internet
**Sync automático en ≤10 segundos tras detectar red.**
- NetworkCallback para validación de red

### RNF-AVAIL-004 [P1] Failover regional
Multi-region deployment (us-east-1 + us-west-2)
- **Post-MVP**

## 4. Compliance y regulatorio

### RNF-COMP-001 [P0] Cumplimiento SUDEBAN
ViCheck opera bajo licencia de VIPPO que ya tiene permiso de SUDEBAN.
- **Documentación:** delegada a VIPPO
- **Validación:** No requerimos permisos propios, operamos como feature de VIPPO

### RNF-COMP-002 [P0] Cumplimiento SENIAT (impuestos)
- Reporte de operaciones generadas al cliente mensualmente (factura fiscal)
- Compatibilidad con formatos SENIAT para libros contables

### RNF-COMP-003 [P0] KYC/anti-lavado
- KYC básico delegado a VIPPO (cédula, email)
- Límite de CashIn sin KYC completo: $500 USD/mes

### RNF-COMP-004 [P0] Protección de datos personales (LOPDP Venezuela)
- Datos personales encriptados en reposo y tránsito
- Derecho al olvido: borrado de cuenta en 30 días
- Política de privacidad publicada

### RNF-COMP-005 [P0] Compliance Google Play
- Privacy policy URL obligatoria
- Data safety form completado
- Permisos sensibles justificados
- Sin SMS scam patterns

### RNF-COMP-006 [P1] PCI-DSS scope
ViCheck NO almacena PAN de tarjetas (delegado a VIPPO).
- **Scope:** solo procesa tokens CashIn

## 6. Usabilidad y accesibilidad

### RNF-UX-001 [P0] Accesibilidad WCAG 2.1 AA
- Contraste de texto ≥ 4.5:1
- Touch targets ≥ 48dp
- Soporte para screen readers (TalkBack)
- Texto escalable

### RNF-UX-002 [P0] Modo offline visible
- Indicador persistente de estado online/offline
- Mensajes claros sobre qué funciona offline

### RNF-UX-003 [P0] Onboarding ≤ 5 pasos
- Máximo 3 pantallas de slides + registro + verificación

### RNF-UX-004 [P1] Soporte multi-idioma
- Español (default)
- Inglés (internacional)

### RNF-UX-005 [P1] Tutorial contextual
- Coachtips la primera vez que se hace cada acción (NFC, QR, etc.)

## 7. Internacionalización

### RNF-I18N-001 [P1] Multi-moneda
- VES (bolívar) - default
- USD como referencia
- Soporte futuro: COP, MXN, etc.

### RNF-I18N-002 [P1] Timezone
- America/Caracas (UTC-4)

## 8. Observabilidad

### RNF-OBS-001 [P0] Logging estructurado
- Logs en formato JSON
- Correlación de request_id cliente-servidor
- CloudWatch Logs Insights queries predefinidas

### RNF-OBS-002 [P0] Métricas de negocio
- Transacciones por minuto/hora/día
- CashIn total / promedio
- CashOut total / promedio
- Tasa de éxito de sync
- Latencia P50/P95/P99 de cada endpoint

### RNF-OBS-003 [P0] Alertas críticas
- Error rate backend > 5%
- Sync queue crece sin drenar
- CashIn falla > 10%
- Lambda cold start > 5s

### RNF-OBS-004 [P0] Crash reporting mobile
- Crashlytics para Android
- Non-fatal errors reportados con contexto

## 10. Mantenibilidad

### RNF-MAINT-001 [P0] Cobertura de tests
- Backend: ≥ 80% unit tests, ≥ 60% integration
- Android core: ≥ 70% unit tests
- End-to-end: flujos críticos cubiertos

### RNF-MAINT-002 [P0] Lint y formato
- Backend: ruff (lint), black (format), mypy (types)
- Android: ktlint, detekt, ktlintFormat

### RNF-MAINT-003 [P0] CI/CD pipeline
- Tests + lint + build en cada PR
- Deploy a staging automático
- Deploy a producción con approval manual

### RNF-MAINT-004 [P0] Documentación de código
- OpenAPI spec auto-generado (FastAPI)
- KDoc para funciones públicas Android
- README actualizado por servicio

### RNF-MAINT-005 [P1] ADR (Architecture Decision Records)
- Cada decisión arquitectónica documentada en `09-decisions/`

## 11. Escalabilidad

### RNF-SCALE-001 [P0] Multi-tenancy
- 10+ tenants simultáneos en MVP
- 100+ tenants en producción

### RNF-SCALE-002 [P0] Usuarios concurrentes
- 1000 usuarios activos simultáneos en MVP
- 100K usuarios activos en producción (post-MVP)

### RNF-SCALE-003 [P0] Transacciones por día
- 10K tx/día en MVP
- 1M tx/día en producción

### RNF-SCALE-004 [P1] Multi-region
- us-east-1 + us-west-2 activos
- Latency <100ms para usuarios en VE

## 12. Resumen

| Categoría | Total RNF | P0 | P1 |
|---|---|---|---|
| Seguridad | 9 | 7 | 2 |
| Performance | 7 | 6 | 1 |
| Disponibilidad | 4 | 3 | 1 |
| Compliance | 6 | 5 | 1 |
| UX/Accesibilidad | 5 | 3 | 2 |
| i18n | 2 | 0 | 2 |
| Observabilidad | 4 | 4 | 0 |
| Mantenibilidad | 5 | 4 | 1 |
| Escalabilidad | 4 | 3 | 1 |
| **Total** | **46** | **35** | **11** |

---

**Próximo documento:** `04-requirements/user-stories/US-cliente-final.md` — historias de usuario del cliente final.