# ViCheck MVP — Preguntas para VIPPO

Documento para reunión de alineación técnica. Llevar a la primera reunión con equipo VIPPO.

## 1. APIs de VIPPO

### 1.1 Documentación
- ¿Hay documentación OpenAPI / Swagger de las APIs existentes?
- ¿Hay un sandbox / staging environment para pruebas?
- ¿Cuál es el SLA de uptime de las APIs?

### 1.2 CashIn
- ¿Cuál es el endpoint exacto para `/validacion de pago`?
- ¿Qué campos requiere el request?
- ¿Qué formato tiene la respuesta?
- ¿Cómo se notifica el resultado (webhook, polling)?
- ¿Cuál es el timeout esperado?

### 1.3 CashOut
- ¿Cuál es el endpoint para liquidación a banco?
- ¿Qué datos bancarios se requieren?
- ¿Cuál es el horario de liquidación?
- ¿Hay límites por día / por monto?

### 1.4 Cuentas
- ¿Cómo se gestionan las cuentas bancarias del cliente en VIPPO?
- ¿El usuario VIPPO ya tiene cuenta bancaria asociada?
- ¿Cómo se vincula un usuario ViCheck a un usuario VIPPO?

### 1.5 Usuarios
- ¿Existe un endpoint para crear / validar usuarios en VIPPO?
- ¿Hay un sistema de identidad común (SSO)?
- ¿VIPPO maneja ya la verificación KYC?

## 2. Multi-tenancy

### 2.1 Merchant onboarding
- ¿Cómo se da de alta un comercio en VIPPO?
- ¿Hay un proceso manual o se puede automatizar vía API?
- ¿Cuánto tarda el proceso?

### 2.2 Branding
- ¿VIPPO soporta multi-tenant con branding custom?
- ¿Hay APIs para gestionar branding por merchant?

### 2.3 Fees
- ¿Cuál es el fee actual por transacción?
- ¿Es configurable por merchant o fijo?

## 3. Compliance y regulatorio

### 3.1 SUDEBAN
- ¿Qué licencia tiene VIPPO actualmente?
- ¿Necesitamos alguna autorización nueva para operar ViCheck?
- ¿Hay restricciones para NFC/QR payments?

### 3.2 SENIAT
- ¿Cómo reporta VIPPO impuestos actualmente?
- ¿Necesitamos integrarnos al sistema fiscal?
- ¿Hay formatos específicos para pagos NFC?

### 3.3 LOPDP
- ¿Cómo maneja VIPPO la protección de datos personales?
- ¿Hay políticas de privacidad existentes?
- ¿Cómo manejamos derecho al olvido?

## 4. Seguridad

### 4.1 Certificados
- ¿VIPPO usa TLS en todas las APIs?
- ¿Hay certificados de cliente?
- ¿Cuál es el proceso para obtener acceso a producción?

### 4.2 Auditoría
- ¿Hay logs de auditoría existentes?
- ¿Cómo accedemos para investigación de incidentes?
- ¿Hay sistema de detección de fraude actual?

## 5. Operación

### 5.1 Piloto
- ¿Cuál es el primer tenant piloto confirmado?
- ¿Cuántos usuarios se esperan en el piloto?
- ¿Hay métricas de éxito definidas?

### 5.2 Soporte
- ¿Quién es responsable del soporte nivel 1?
- ¿Hay un canal para reportar bugs?
- ¿Cuál es el SLA de respuesta?

### 5.3 Recursos
- ¿Habrá un diseñador UX para la app?
- ¿Habrá un dev adicional para backend?
- ¿Quién hace QA?

## 6. Producto

### 6.1 Alcance MVP
- ¿Qué funcionalidades son MUST-HAVE vs NICE-TO-HAVE?
- ¿El piloto requiere iOS?
- ¿Hay un deadline duro para MVP?

### 6.2 Casos de uso
- ¿El piloto incluye transporte (La Bandera)?
- ¿Qué tenant piloto es primero?
- ¿Cuál es el timeline realista?

## 7. Presupuesto

### 7.1 Infraestructura
- ¿Hay presupuesto aprobado para AWS?
- ¿Hay límites de gasto mensual?
- ¿Quién aprueba nuevos servicios?

### 7.2 Equipo
- ¿Hay presupuesto para contratar más devs?
- ¿Hay freelancers disponibles?

## 8. Despliegue

### 8.1 Google Play
- ¿Hay cuenta de Google Play existente para VIPPO?
- ¿Hay políticas internas de release?

### 8.2 Backend
- ¿Se aprueba AWS SAM + Lambda Python 3.14?
- ¿Hay instancias existentes para integración?

---

**Próximo documento:** `09-decisions/ADR-001-multiplatform.md`