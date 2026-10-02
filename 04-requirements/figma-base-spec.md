# ViCheck MVP — Figma Base (Wireframes y Design Spec)

> **Documento de referencia para arrancar el diseño en Figma.** NO lee especificaciones funcionales completas.

## 1. Información para el diseñador

**Producto:** ViCheck — billetera móvil offline-first para Venezuela
**Audiencia:** Clientes finales (consumidores) + Prestadores (comercios)
**Tipo:** App Android nativa (Kotlin + Jetpack Compose)
**Estado:** MVP — closed beta
**Estilo visual:** Moderno, confiable, fintech (similar a Nequi, Yape, Mercado Pago)

## 2. Sistema de diseño base

### 2.1 Paleta de colores

```
PRIMARIO (confianza, fintech):
  Primary 900:  #003D5C  (azul marino profundo)
  Primary 700:  #006090  (botones, CTAs)
  Primary 500:  #0099D9  (acentos, highlights)
  Primary 100:  #E0F2FA  (backgrounds sutiles)

SECUNDARIO (offline, premium):
  Secondary 900: #2D1B4E  (morado oscuro)
  Secondary 500: #6B4FBB  (acento de marca)
  Secondary 100: #F0E9FB

ESTADO:
  Success:  #10B981  (verde, trans OK)
  Error:    #EF4444  (rojo, error)
  Warning:  #F59E0B  (naranja, alerta)
  Info:     #3B82F6  (azul info)

NEUTRALES:
  White:    #FFFFFF
  Gray 50:  #F9FAFB
  Gray 100: #F3F4F6
  Gray 200: #E5E7EB
  Gray 400: #9CA3AF
  Gray 600: #4B5563
  Gray 800: #1F2937
  Gray 900: #111827
  Black:    #000000
```

### 2.2 Tipografía

```
FUENTE: Inter (sans-serif moderno)

Display:  Inter Bold 32px
H1:       Inter Bold 24px
H2:       Inter SemiBold 20px
H3:       Inter Medium 18px
Body L:   Inter Regular 16px
Body M:   Inter Regular 14px
Body S:   Inter Regular 12px
Caption:  Inter Medium 11px

NÚMEROS (balances):
  Inter Bold 32px (balance principal)
  Inter SemiBold 16px (montos en lista)
  Monospace para IDs (JetBrains Mono)
```

### 2.3 Iconografía

- **Set:** Material Icons + Phosphor Icons
- **Estilo:** Outline, 24px default
- **Iconos críticos:**
  - NFC tap (icono de ondas)
  - QR scanner (cámara)
  - Wallet / saldo (tarjeta)
  - Sync (flechas circulares)
  - CashIn (flecha hacia abajo)
  - CashOut (flecha hacia arriba)
  - Pay (transferencia)
  - Settings (engranaje)

### 2.4 Espaciado

```
Base: 4px

Espacios:
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 32px
  2xl: 48px

Radio de bordes:
  sm: 4px
  md: 8px
  lg: 16px
  xl: 24px
  full: 9999px
```

## 3. Pantallas del MVP (ordenadas por flujo)

### 3.1 Flujo cliente final

#### [C-01] Splash / Carga inicial
**Pantalla negra con logo ViCheck, 2s de duración.**

#### [C-02] Onboarding (3 slides)
```
Slide 1: "Sin señal. Ni problema."
         - icono de conectividad tachada
         - "Paga incluso sin internet"

Slide 2: "Acercar y listo"
         - ilustración de NFC tap
         - "Acerca tu pulsera o celular"

Slide 3: "Seguro siempre"
         - icono de escudo
         - "Tus fondos protegidos con criptografía"
```

Botón: "Empezar" (CTA grande)

#### [C-03] Login / Registro
**Layout:**
- Logo ViCheck (top)
- Input: Teléfono (con código país +58)
- Botón: "Continuar"
- Link: "Ya estoy registrado"

#### [C-04] Verificación SMS
- Input: código de 6 dígitos
- Botón: "Verificar"
- Botón secundario: "Reenviar código"
- Countdown de 30s

#### [C-05] Onboarding datos personales
- Input: Nombre completo
- Input: Cédula
- Input: Email (opcional)
- Botón: "Siguiente"

#### [C-06] Selección de banco origen (CashIn)
- Lista de bancos VE afiliados
- Input: número de cuenta
- Botón: "Continuar"

#### [C-07] Home / Billetera
**Layout principal:**
```
┌─────────────────────────────────────┐
│  [avatar] Hola, Francisco    [⚙️]   │
│                                      │
│  ╔══════════════════════════════╗   │
│  ║   Mi saldo                    ║   │
│  ║   Bs. 12,500.00              ║   │
│  ║   [+ Recargar] [↗ Enviar]    ║   │
│  ╚══════════════════════════════╝   │
│                                      │
│  ┌──────────┐ ┌──────────┐         │
│  │ [📡 NFC] │ │ [📷 QR] │          │
│  │ Pagar    │ │ Escanear │         │
│  └──────────┘ └──────────┘         │
│                                      │
│  Actividad reciente                  │
│  ┌──────────────────────────────┐  │
│  │ Hoy, 14:32                   │  │
│  │ Pago en La Bandera   -Bs.45  │  │
│  │ ✓ Confirmado                  │  │
│  ├──────────────────────────────┤  │
│  │ Hoy, 09:15                   │  │
│  │ Recarga recibida  +Bs.5000  │  │
│  │ ✓ Confirmado                  │  │
│  └──────────────────────────────┘  │
│                                      │
│  [Home]  [Historial]  [Más]          │
└─────────────────────────────────────┘
```

#### [C-08] CashIn / Recargar
- Monto predefinidos: Bs. 500, 1000, 2000, 5000
- Input custom de monto
- Selector de banco origen
- Resumen de la operación
- Botón: "Confirmar recarga"

#### [C-09] CashIn en proceso
- Indicador de carga
- Texto: "Conectando con tu banco..."
- Botón disabled

#### [C-10] CashIn exitoso
- Icono de checkmark
- Texto: "¡Recarga exitosa!"
- Monto: Bs. 5000.00
- Nuevo balance: Bs. 17,500.00
- Botón: "Volver al inicio"

#### [C-11] CashIn fallido
- Icono de error
- Texto: "No pudimos procesar la recarga"
- Razón: <detalle del error>
- Botones: "Reintentar" | "Volver"

#### [C-12] Pagar con NFC
**Pantalla de "acerca tu tag":**
```
┌─────────────────────────────────────┐
│              ← Volver                │
│                                      │
│         [icono NFC animado]          │
│                                      │
│      Acerca tu pulsera o tarjeta     │
│         al POS del comercio          │
│                                      │
│         Esperando conexión...        │
│                                      │
│     ┌────────────────────────┐     │
│     │ Bs. 12,500.00   [tu balance] │ │
│     └────────────────────────┘     │
└─────────────────────────────────────┘
```

#### [C-13] Confirmar pago (con PIN)
```
┌─────────────────────────────────────┐
│              ← Cancelar            │
│                                      │
│   Pago a Terminal La Bandera         │
│   Bs. 45.00                          │
│                                      │
│   Ingresa tu PIN para confirmar:    │
│                                      │
│   ┌─┐  ┌─┐  ┌─┐  ┌─┐  ┌─┐  ┌─┐    │
│   │●│  │●│  │●│  │ │  │ │  │ │    │
│   └─┘  └─┘  └─┘  └─┘  └─┘  └─┘    │
│                                      │
│   Nuevo balance: Bs. 12,455.00        │
└─────────────────────────────────────┘
```

#### [C-14] Pago exitoso
- Animación de checkmark
- Texto: "¡Pago exitoso!"
- Detalles: comercio, monto, nuevo balance
- Botones: "Ver recibo" | "Volver al inicio"

#### [C-15] Pagar con QR
- Cámara activa con marco de escaneo
- "Escanea el código QR del comercio"

#### [C-16] Historial completo
- Lista paginada de transacciones
- Filtros: por fecha, monto, tipo

#### [C-17] Detalle de transacción
```
┌─────────────────────────────────────┐
│         ← Volver                      │
│                                      │
│       ✓ Confirmado                    │
│   Bs. 45.00                           │
│   Pago en Terminal La Bandera        │
│                                      │
│   ID transacción:                    │
│   a3f7c8b2-...                       │
│                                      │
│   Fecha: 2 oct 2026, 14:32          │
│   Tipo: Pago a comercio             │
│   Comercio: La Bandera            │
│                                      │
│   Firma digital:                    │
│   0x9f4a...                         │
│                                      │
│   [Compartir recibo] [Soporte]      │
└─────────────────────────────────────┘
```

#### [C-18] Settings
- Mi perfil (nombre, cédula, email)
- Seguridad (cambiar PIN, biometría)
- NFC (gestionar tags emparejados)
- Notificaciones
- Privacidad
- Cerrar sesión

#### [C-19] Emparejar tag NFC
- Input: nombre del tag
- "Acerca el tag NFC"
- Animación de ondas cuando lo detecta
- Confirmación + PIN
- Botón: "Emparejado exitosamente"

### 3.2 Flujo POS comercio

#### [P-01] POS Login
- Tenant ID
- Usuario POS
- PIN

#### [P-02] POS Home / Cobrar
```
┌─────────────────────────────────────┐
│  Terminal La Bandera    [usuario ▼]  │
│                                      │
│  ╔══════════════════════════════╗   │
│  ║   Cobrar ahora               ║   │
│  ║                              ║   │
│  ║   Bs. ____________           ║   │
│  ║                              ║   │
│  ║  [📡 NFC] [📷 QR] [✋ Manual] ║   │
│  ╚══════════════════════════════╝   │
│                                      │
│  Ventas hoy                          │
│   Bs. 12,450.00 / 87 transacciones   │
│                                      │
│  Estado: ● Online   Sync: 0 pend.   │
└─────────────────────────────────────┘
```

#### [P-03] POS esperando tap NFC
```
┌─────────────────────────────────────┐
│        ← Cancelar                    │
│                                      │
│       [icono NFC animado]            │
│                                      │
│   Acerca el tag del cliente          │
│                                      │
│   Monto: Bs. 45.00                  │
│                                      │
│   Esperando tag...                   │
│                                      │
│   Cliente UID: 04:A3:B2:...         │
└─────────────────────────────────────┘
```

#### [P-04] POS esperando QR
- Cámara activa
- "Cliente escanea QR"

#### [P-05] POS confirmando transacción
- "Procesando..."
- Detalles de la tx

#### [P-06] POS venta exitosa
- Animación de checkmark
- "¡Venta exitosa!"
- Detalles: cliente UID, monto, hora
- Botones: "Nueva venta" | "Recibo"

#### [P-07] POS saldo insuficiente
- "El cliente no tiene saldo suficiente"
- Saldo actual del cliente
- Botón: "OK"

#### [P-08] POS reportar problema
- Selección de categoría
- Input: descripción
- Botón: "Enviar"

#### [P-09] POS sync
- Indicador "Sincronizando X transacciones..."
- Barra de progreso
- Resultado: "✓ X transacciones sincronizadas"

#### [P-10] POS dashboard reportes
- KPIs: ventas hoy/semana/mes
- Gráfico de barras de ventas por día
- Lista de transacciones

#### [P-11] POS CashOut
- Balance disponible
- Input: monto a liquidar
- Botón: "Solicitar liquidación"

### 3.3 Flujo Admin VIPPO (futuro P2)

#### [A-01] Admin login
- Email + password + 2FA

#### [A-02] Admin dashboard
- KPIs globales
- Mapa de calor de actividad
- Lista de alertas

#### [A-03] Lista de tenants
- Tabla con filtros

#### [A-04] Detalle de tenant
- Info del comercio
- Balance
- Transacciones
- Configuración

#### [A-05] Detalle de transacción (cross-tenant)
- Vista detallada
- Acciones: reversar, congelar, escalar

## 4. Componentes reutilizables (Design System)

Lista de componentes que el diseñador debe entregar antes de empezar las pantallas:

### Botones
- Primary Button (Filled)
- Secondary Button (Outlined)
- Tertiary Button (Text)
- Destructive Button (Filled Red)
- Button with Icon (left/right)
- Button Loading State

### Inputs
- Text Input (default, focused, error, disabled)
- Phone Input (con país)
- Currency Input (con prefijo Bs.)
- PIN Input (6 dígitos)
- OTP Input
- Search Input

### Cards
- Balance Card (con CTA actions)
- Transaction Card (en historial)
- Info Card (banner informativo)
- Error Card
- Success Card

### Modales / Sheets
- Bottom Sheet genérico
- Modal de confirmación
- Modal de error
- Action Sheet (acciones múltiples)

### Navigation
- Bottom Navigation (3-5 items)
- Top App Bar (con back, título, acciones)
- Tabs (segmentados)

### Status
- Online/Offline indicator (chip)
- Sync status badge
- Loading spinner
- Empty state component
- Error state component

### Charts
- Bar chart (ventas por día)
- Line chart (balance evolución)

## 5. Pantallas priorizadas para Figma (orden)

1. **[C-07] Home / Billetera** (la más importante)
2. **[C-12] Pagar con NFC** (core de la propuesta)
3. **[C-13] Confirmar pago con PIN** (crítico)
4. **[C-14] Pago exitoso** (feedback)
5. **[C-08] CashIn / Recargar** (entrada de dinero)
6. **[P-02] POS Home** (comercio)
7. **[P-03] POS esperando tap** (comercio)
8. **[P-06] POS venta exitosa** (feedback)
9. **[C-02] Onboarding 3 slides** (primera impresión)
10. **[C-03] Login / Registro** (entrada)
11. **[C-17] Detalle de transacción** (auditoría)
12. Resto en orden de flujo

## 6. Tokens de diseño (Figma Variables)

```
Color/Primary/900 → #003D5C
Color/Primary/700 → #006090
Color/Primary/500 → #0099D9
Color/Primary/100 → #E0F2FA
...
Color/Secondary/900 → #2D1B4E
...
Color/State/Success → #10B981
...
Color/Neutral/White → #FFFFFF
Color/Neutral/Gray-50 → #F9FAFB
...

Spacing/xs → 4px
Spacing/sm → 8px
Spacing/md → 16px
Spacing/lg → 24px
Spacing/xl → 32px

Radius/sm → 4px
Radius/md → 8px
Radius/lg → 16px
Radius/xl → 24px
Radius/full → 9999px

Typography/Display → Inter Bold 32px
Typography/H1 → Inter Bold 24px
...
```

## 7. Consideraciones especiales

### 7.1 Modo offline
- Las pantallas deben mostrar claramente **estado online/offline**
- Indicador persistente en top bar
- Mensajes contextuales: "Estás offline. La transacción se guardará y sincronizará cuando vuelvas online."

### 7.2 Multi-tenancy visual
- Cada tenant puede customizar:
  - Color primario (Primary/700)
  - Logo
  - Nombre comercial
- Componentes deben usar tokens, no colores hardcoded

### 7.3 Accesibilidad
- Contraste WCAG AA mínimo
- Touch targets ≥ 48dp
- Text escalable (soporte dynamic type)

### 7.4 Modo oscuro
- Soporte nativo Android
- Variables claras para modo oscuro
- Grays se invierten, primarios se ajustan

## 8. Recursos para el diseñador

- **Estilo visual:** Nequi (Colombia), Yape (Perú), Mercado Pago (Latam), Cash App (US)
- **Iconografía:** Phosphor Icons (https://phosphoricons.com/) — open source
- **Inspiración fintech:** ver capturas reales de Yape + Nequi + Mercado Pago Wallet
- **NFC interaction:** ver videos de Apple Pay / Google Pay tap animations

## 9. Entregables del Figma

1. **Cover** (portada del archivo con descripción del producto)
2. **Design System** (componentes + variables + estilos)
3. **Onboarding flow** (C-01 a C-06)
4. **Wallet flow** (C-07 a C-17)
5. **Settings flow** (C-18, C-19)
6. **POS flow** (P-01 a P-11)
7. **Estados vacíos** (sin saldo, sin transacciones, sin conexión)
8. **Estados de error** (todos los flujos)
9. **Modo oscuro** (variantes de pantallas clave)
10. **Handoff para Android** (especificaciones de implementación)

---

**Próximo documento:** `04-requirements/non-functional.md` — requisitos no funcionales.