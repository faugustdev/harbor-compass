# ViCheck MVP — Especificación Completa de Flujos UI

> **Documento de referencia para el diseño en Figma y para la implementación en Android/Web.** Cubre el 100% de los flujos de usuario (cliente final), aliado comercial (POS comercio + dashboard admin del aliado) y administrador ViPPO (web admin).

## Convenciones

- `**[C-XX]**` = Pantalla de Cliente final (móvil)
- `**[P-XX]**` = Pantalla de POS comercio (móvil, modo POS)
- `**[D-XX]**` = Pantalla de Dashboard del aliado (móvil, modo gerente)
- `**[W-XX]**` = Pantalla de Web Admin ViPPO (escritorio)
- `**[F-XX]**` = Flujo completo (secuencia de pantallas)

**Estados de pantalla:**
- `[Onb]` = Onboarding
- `[Auth]` = Autenticación
- `[Main]` = Pantalla principal
- `[Action]` = Acción en curso
- `[Confirm]` = Confirmación requerida
- `[Success]` = Resultado exitoso
- `[Error]` = Resultado fallido
- `[Empty]` = Sin datos
- `[Offline]` = Sin conectividad
- `[Loading]` = Cargando

**Tokens visuales (de figma-base-spec.md):**
- `Primary 700: #006090` — CTAs principales
- `Secondary 500: #6B4FBB` — Accento offline / marca
- `Success: #10B981` — Transacciones OK
- `Error: #EF4444` — Errores
- `Warning: #F59E0B` — Alertas
- `Gray 600: #4B5563` — Texto secundario
- `Gray 100: #F3F4F6` — Backgrounds sutiles

**Símbolos de medios de pago:**
- 📡 NFC (chip a teléfono)
- 📷 QR (cámara)
- 📲 Celular como tag (HCE)
- ✋ Manual (sin medio físico)

---

# ÍNDICE DE FLUJOS

## Cliente final (C)
- [F-C-01] Onboarding y registro de cuenta
- [F-C-02] Conectar dispositivo NFC (pulsera, tarjeta, celular)
- [F-C-03] CashIn / Recargar saldo
- [F-C-04] CashOut / Retirar saldo a banco
- [F-C-05] Pago NFC (chip a teléfono)
- [F-C-06] Pago QR (cámara)
- [F-C-07] Pago phone-to-phone P2P
- [F-C-08] Pago con el mismo celular (HCE)
- [F-C-09] Pago sin conexión (fallback universal)
- [F-C-10] Ver historial y detalle de transacción
- [F-C-11] Configuración de seguridad (PIN, biometría)
- [F-C-12] Notificaciones
- [F-C-13] Reembolsos y disputas
- [F-C-14] Recuperación de cuenta
- [F-C-15] Eliminar cuenta
- [F-C-16] Promoción y cashback

## Aliado / Comercio (D = Dashboard aliado, P = POS)
- [F-A-01] Onboarding del aliado
- [F-A-02] Configurar branding y fees del tenant
- [F-A-03] Cobro NFC (POS)
- [F-A-04] Cobro QR (POS)
- [F-A-05] Cobro manual (POS)
- [F-A-06] Sync de transacciones (POS)
- [F-A-07] CashOut / Liquidación a banco del aliado
- [F-A-08] Dashboard de ventas (móvil gerente)
- [F-A-09] Gestión de empleados (multi-tenant POS)
- [F-A-10] Notificación de cashin recibido del cliente
- [F-A-11] Gestión de promociones

## Web Admin ViPPO (W)
- [F-W-01] Login admin + 2FA
- [F-W-02] Ver lista de aliados (tenants)
- [F-W-03] Crear/editar/deshabilitar aliado
- [F-W-04] Ver lista de usuarios
- [F-W-05] CRUD de datos de usuario
- [F-W-06] Verificaciones de seguridad y KYC
- [F-W-07] Anti-fraude: revisar tx sospechosas
- [F-W-08] Anti-fraude: congelar wallet
- [F-W-09] Anti-fraude: reversar transacción
- [F-W-10] Conciliación manual
- [F-W-11] Reportes regulatorios (SENIAT)
- [F-W-12] Configuración global de fees y límites

---

# PARTE 1 — CLIENTE FINAL

## F-C-01: Onboarding y registro de cuenta

**Pantallas:** C-01, C-02, C-03, C-04, C-05

```
[C-01 Splash 2s] → [C-02 Onboarding 3 slides] → [C-03 Login/Registro]
                        ↓
                [C-04 Verificación SMS OTP]
                        ↓
              [C-05 Onboarding datos personales]
                        ↓
              [C-06 Selección banco origen]
                        ↓
                 [C-07 Home / Billetera]
```

**Estados clave:**
- Si OTP falla 3 veces → bloqueo temporal 5 min
- Si KYC incompleto → CashIn limitado a $500/mes
- Si banco no afiliado → mensaje "Banco no soportado, prueba con otro"

**Pantallas detalladas:**

#### [C-01] Splash / Carga inicial
```
┌─────────────────────────────────────┐
│                                     │
│                                     │
│          [Logo ViCheck]             │
│                                     │
│                                     │
│                                     │
└─────────────────────────────────────┘
```
- Fondo: Primary 900 (#003D5C)
- Logo: blanco, centrado
- Duración: 2 segundos

#### [C-02] Onboarding (3 slides)
```
┌─────────────────────────────────────┐
│ [Skip]                              │
│                                     │
│      [Ilustración NFC + offline]    │
│                                     │
│      Sin señal. Ni problema.        │
│                                     │
│  Paga incluso sin internet, tu     │
│  dinero siempre seguro.            │
│                                     │
│     ● ○ ○    [Empezar →]            │
└─────────────────────────────────────┘
```

- 3 slides (ya documentadas en figma-base-spec)
- Botón "Saltar" arriba a la derecha
- Indicador de progreso (dots)
- Botón CTA primario: "Empezar" o "Siguiente"

#### [C-03] Login / Registro
```
┌─────────────────────────────────────┐
│ [←]                                 │
│                                     │
│      [Logo ViCheck]                │
│                                     │
│   Ingresa tu teléfono              │
│                                     │
│  🇻🇪 +58 [____________________]    │
│                                     │
│  [   Continuar   ]                 │
│                                     │
│  ¿Ya tienes cuenta? Inicia sesión  │
│                                     │
└─────────────────────────────────────┘
```

- Selector de país (default VE)
- Input con máscara automática
- Validación: solo números, 10 dígitos VE
- Botón deshabilitado hasta validar formato

#### [C-04] Verificación SMS
```
┌─────────────────────────────────────┐
│ [←]                                 │
│                                     │
│   Te enviamos un código al         │
│   +58 412 1234567                  │
│                                     │
│   ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐         │
│   │ │ │ │ │ │ │ │ │ │ │ │         │
│   └─┘ └─┘ └─┘ └─┘ └─┘ └─┘         │
│                                     │
│   Reenviar código (00:28)          │
│                                     │
│   [   Verificar   ]                │
└─────────────────────────────────────┘
```

- 6 dígitos individuales
- Auto-focus siguiente input
- Countdown 30s para reenvío
- Error: shake animation + texto rojo

#### [C-05] Datos personales
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Cuéntanos sobre ti               │
│                                     │
│   Nombre completo                   │
│   [____________________]            │
│                                     │
│   Cédula                            │
│   [V-_________]                     │
│                                     │
│   Email (opcional)                  │
│   [____________________]            │
│                                     │
│   [   Continuar   ]                │
└─────────────────────────────────────┘
```

- Validación KYC automática con SENIAT (SUDEBAN no aplica — no custodiamos dinero)
- Email opcional
- Si KYC falla → mensaje "Verifica tu cédula"

#### [C-06] Selección banco origen
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Selecciona tu banco              │
│                                     │
│   🔍 [Buscar banco...]            │
│                                     │
│   ┌────────────────────┐          │
│   │ 🏦 Banesco        │          │
│   ├────────────────────┤          │
│   │ 🏦 Mercantil      │          │
│   ├────────────────────┤          │
│   │ 🏦 Provincial     │          │
│   ├────────────────────┤          │
│   │ 🏦 Venezuela      │          │
│   ├────────────────────┤          │
│   │ 🏦 BNC            │          │
│   └────────────────────┘          │
│                                     │
│   [   Continuar   ]                │
└─────────────────────────────────────┘
```

- Lista de bancos VIPPO afiliados
- Búsqueda con filtro live
- Cache de selección

---

## F-C-02: Conectar dispositivo NFC (pulsera, tarjeta, celular)

**Pantallas:** C-19 + nuevas: [C-20], [C-22], [C-23]

### Variantes del flujo:

**Variante A: Emparejar pulsera/tarjeta**
```
[Settings → NFC → Emparejar nuevo tag]
              ↓
       [C-19 Emparejar tag NFC]
              ↓
       (acercar tag)
              ↓
       [C-20 Confirmar con PIN]
              ↓
       [C-21 Emparejado exitoso]
```

**Variante B: Emparejar celular (HCE)**
```
[Settings → NFC → Usar este celular como tag]
              ↓
       [C-22 Activar modo celular]
              ↓
       (instalar perfil ViCheck)
              ↓
       [C-23 Celular activado como tag]
```

**Variante C: Transferir dispositivo (cambio de celular o pulsera)**
```
[Settings → NFC → Mis dispositivos]
              ↓
       [C-24 Lista de dispositivos emparejados]
              ↓
       [C-25 Transferir / Desemparejar]
              ↓
       [Confirmar con PIN]
              ↓
       [C-26 Transferencia completada]
```

### Pantallas detalladas:

#### [C-19] Emparejar tag NFC (pulsera/tarjeta)
```
┌─────────────────────────────────────┐
│ [←]                              │
│                                     │
│      [Animación ondas NFC]         │
│                                     │
│      Acerca tu pulsera o tarjeta  │
│                                     │
│      Detectando...                 │
│                                     │
│   ┌────────────────────────┐      │
│   │ Tags detectados:          │      │
│   │ UID 04:A3:B2:C1:D4:E5:F6│      │
│   └────────────────────────┘      │
│                                     │
│      Nombre del tag:               │
│      [____________________]        │
│                                     │
│      [   Emparejar   ]            │
└─────────────────────────────────────┘
```

- Animación de ondas pulsantes
- Lista de tags cercanos (puede haber varios)
- Selector si múltiples tags
- Nombre opcional

#### [C-20] Confirmar con PIN para emparejar
```
┌─────────────────────────────────────┐
│ [← Cancelar]                        │
│                                     │
│   Confirma con tu PIN               │
│                                     │
│   ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐      │
│   │●│ │●│ │●│ │ │ │ │ │ │      │
│   └─┘ └─┘ └─┘ └─┘ └─┘ └─┘      │
│                                     │
│   Emparejando: "Mi pulsera gym"    │
│   UID: 04:A3:B2:C1:D4:E5:F6        │
│                                     │
│   [Huella digital]                  │
└─────────────────────────────────────┘
```

- 6 dígitos o biometría
- Si falla 5 veces → bloqueo 5 min
- Mensaje de confirmación clara

#### [C-21] Emparejado exitoso
```
┌─────────────────────────────────────┐
│                                     │
│         ✓                          │
│                                     │
│   ¡Listo! Tu tag está emparejado   │
│                                     │
│   "Mi pulsera gym"                  │
│                                     │
│   UID: 04:A3:B2:C1:D4:E5:F6        │
│                                     │
│   Ahora puedes pagar con él en     │
│   cualquier comercio afiliado.      │
│                                     │
│   [ Emparejar otro ] [ Listo ]    │
└─────────────────────────────────────┘
```

#### [C-22] Activar celular como tag (HCE)
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   📲 Activar este celular como tag  │
│                                     │
│   Esto te permite pagar sin          │
│   sacar efectivo ni llevar tarjetas. │
│   Tu celular emitirá una señal NFC  │
│   como si fuera una tarjeta.       │
│                                     │
│   Requisitos:                       │
│   ✓ Android 7.0 o superior         │
│   ✓ NFC habilitado                │
│   ✓ Pantalla encendida           │
│                                     │
│   [   Activar   ]                  │
└─────────────────────────────────────┘
```

#### [C-23] Celular activado como tag
```
┌─────────────────────────────────────┐
│                                     │
│         ✓                          │
│                                     │
│   ¡Tu celular es ahora un tag NFC!│
│                                     │
│   📲                                │
│                                     │
│   Para pagar:                       │
│   1. Desbloquea la pantalla        │
│   2. Acerca al POS                 │
│   3. Confirma con tu PIN          │
│                                     │
│   [ Probar ahora ] [ Configuración ]│
└─────────────────────────────────────┘
```

#### [C-24] Mis dispositivos emparejados
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Mis dispositivos NFC              │
│                                     │
│   ┌────────────────────────┐      │
│   │ 📲 Este celular       │      │
│   │    Activo             │      │
│   ├────────────────────────┤      │
│   │ ⌚ Mi pulsera gym     │      │
│   │    UID 04:A3:...      │      │
│   │    Activo · gym empaque││
│   ├────────────────────────┤      │
│   │ 💳 Tarjeta oficina    │      │
│   │    UID 04:B2:...      │      │
│   │    Activo             │      │
│   └────────────────────────┘      │
│                                     │
│   [+ Emparejar nuevo]              │
│                                     │
└─────────────────────────────────────┘
```

- Lista con iconos por tipo (📲 ⌚ 💳)
- Estado: Activo / Inactivo
- Cada item: editar nombre, desemparejar

#### [C-25] Desemparejar / Transferir
```
┌─────────────────────────────────────┐
│ [← Cancelar]                        │
│                                     │
│   ⌚ "Mi pulsera gym"               │
│   UID: 04:A3:B2:C1:D4:E5:F6        │
│                                     │
│   ⚠️ Al desemparejar este tag, ya │
│   不会再 podrás pagar con él.        │
│                                     │
│   Opciones:                         │
│   ○ Desemparejar (queda inutilizable)│
│   ○ Transferir a otra persona     │
│                                     │
│   Ingresa tu PIN para confirmar:  │
│   ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐       │
│   │ │ │ │ │ │ │ │ │ │ │ │       │
│   └─┘ └─┘ └─┘ └─┘ └─┘ └─┘       │
│                                     │
│   [   Confirmar   ]                │
└─────────────────────────────────────┘
```

---

## F-C-03: CashIn / Recargar saldo

**Pantallas:** C-08, C-09, C-10, C-11 + nueva [C-08b confirmación]

### Flujo completo:

```
[Home] → [C-08 Seleccionar monto] → [C-08b Confirmar recarga]
                                            ↓
                                     [C-09 Procesando]
                                            ↓
                              ┌─────────────┴────────────┐
                              ↓                          ↓
                       [C-10 Exitoso]            [C-11 Fallido]
```

#### [C-08] Seleccionar monto de recarga
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Recargar saldo                    │
│                                     │
│   Saldo actual: Bs. 12,500.00      │
│                                     │
│   Monto rápido:                     │
│   ┌────────┐ ┌────────┐            │
│   │ Bs.500 │ │Bs.1,000│            │
│   └────────┘ └────────┘            │
│   ┌────────┐ ┌────────┐            │
│   │Bs.2,000│ │Bs.5,000│            │
│   └────────┘ └────────┘            │
│                                     │
│   O ingresa otro monto:            │
│   Bs. [____________________]       │
│                                     │
│   Banco origen:                     │
│   🏦 Banesco - ****1234   [Cambiar]│
│                                     │
│   💡 Recibirás un push al           │
│   completarse la operación.        │
│                                     │
│   [   Continuar   ]                │
└─────────────────────────────────────┘
```

- Montos predeterminados: 500, 1000, 2000, 5000
- Monto custom con teclado numérico
- Validación: mín 100 VES, máx diario
- Si excede límite → mensaje contextual

#### [C-08b] Confirmar recarga
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Confirma tu recarga              │
│                                     │
│   ┌────────────────────────┐      │
│   │                          │      │
│   │   Bs. 2,000.00           │      │
│   │                          │      │
│   │   Desde Banesco ****1234 │      │
│   │                          │      │
│   │   Comisión: Bs. 0        │      │
│   │                          │      │
│   │   ⏱️ Tarda unos 5 seg.   │      │
│   │                          │      │
│   └────────────────────────┘      │
│                                     │
│   Confirma con tu PIN:              │
│   ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐       │
│   │ │ │ │ │ │ │ │ │ │ │ │       │
│   └─┘ └─┘ └─┘ └─┘ └─┘ └─┘       │
│                                     │
│   [   Confirmar recarga   ]                  │
└─────────────────────────────────────┘
```

- Resumen visual claro
- PIN obligatorio (RNF-SEC-002)
- Botón deshabilitado hasta PIN completo

#### [C-09] Procesando
```
┌─────────────────────────────────────┐
│                                     │
│         [Spinner animado]           │
│                                     │
│      Conectando con tu banco...    │
│                                     │
│      ⏱️ ~5 segundos                │
│                                     │
│   ┌────────────────────────┐      │
│   │ Bs. 2,000.00            │      │
│   │ Banesco ****1234        │      │
│   └────────────────────────┘      │
│                                     │
│   [   Cancelar   ] (con timer)     │
└─────────────────────────────────────┘
```

- Spinner + texto tranquilizador
- Cancelar solo primeros 10s

#### [C-10] Recarga exitosa
```
┌─────────────────────────────────────┐
│                                     │
│         ✓ (animado)                │
│                                     │
│      ¡Recarga exitosa!             │
│                                     │
│   Bs. 2,000.00                     │
│                                     │
│   Saldo anterior:    Bs. 12,500.00│
│   + Recarga:         Bs.  2,000.00│
│   ─────────────────────────────   │
│   Saldo nuevo:       Bs. 14,500.00│
│                                     │
│   🕐 Hace 2 segundos              │
│   📋 ID: cashin-uuid-corto        │
│                                     │
│   [ Ver historial ] [ Inicio ]   │
└─────────────────────────────────────┘
```

#### [C-11] Recarga fallida
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│         ✗                          │
│                                     │
│      No pudimos procesar la       │
│      recarga                        │
│                                     │
│   Razón:                           │
│   "Saldo insuficiente en tu       │
│    cuenta bancaria"                │
│                                     │
│   Bs. 2,000.00 - Banesco ****1234 │
│                                     │
│   💡 Verifica tu saldo bancario.  │
│                                     │
│   [   Reintentar   ] [ Inicio ]   │
└─────────────────────────────────────┘
```

---

## F-C-04: CashOut / Retirar saldo a banco

**Pantallas:** nuevas [C-30] a [C-34]

```
[Home] → [C-30 Seleccionar monto CashOut]
            ↓
       [C-31 Confirmar CashOut]
            ↓
       [C-32 Procesando CashOut]
            ↓
    ┌───────┴───────┐
    ↓               ↓
[C-33 Exitoso] [C-34 Fallido]
```

#### [C-30] Seleccionar monto CashOut
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Retirar a mi banco                │
│                                     │
│   Disponible: Bs. 12,500.00       │
│   (No incluye saldo pendiente sync)│
│                                     │
│   Retirar todo: Bs. 12,500.00    │
│                                     │
│   O monto custom:                         │
│   Bs. [____________________]       │
│                                     │
│   Banco destino:                     │
│   🏦 Banesco - ****1234            │
│                                     │
│   ⏱️ Llega en 24 horas hábiles      │
│                                     │
│   [   Continuar   ]                │
└─────────────────────────────────────┘
```

- "Retirar todo" por defecto
- Validación: no exceder disponible
- Tiempo estimado: "Llega en 24h hábiles"

#### [C-31] Confirmar CashOut
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Confirma tu retiro                │
│                                     │
│   ┌────────────────────────┐      │
│   │   Bs. 12,500.00         │      │
│   │   A Banesco ****1234   │      │
│   │   ⏱️ 24h hábiles        │      │
│   └────────────────────────┘      │
│                                     │
│   PIN:                              │
│   ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐       │
│   │ │ │ │ │ │ │ │ │ │ │ │       │
│   └─┘ └─┘ └─┘ └─┘ └─┘ └─┘       │
│                                     │
│   [   Confirmar retiro   ]                  │
└─────────────────────────────────────┘
```

#### [C-32] Procesando CashOut
```
[Spinner + "Procesando tu retiro..."]

Llega en 24h hábiles
```

#### [C-33] CashOut exitoso
```
┌─────────────────────────────────────┐
│                                     │
│         ✓                          │
│                                     │
│      ¡Retiro iniciado!             │
│                                     │
│   Bs. 12,500.00                     │
│   A Banesco ****1234             │
│                                     │
│   💡 Llega en 24h hábiles.         │
│      Te notificaremos cuando       │
│      se acredite.                  │
│                                     │
│   📋 ID: cashout-uuid-corto       │
│                                     │
│   [ Ver historial ] [ Inicio ]   │
└─────────────────────────────────────┘
```

---

## F-C-05: Pago NFC (chip a teléfono)

**Pantallas:** C-12, C-13, C-14

```
[Home] → [C-12 Esperar tap NFC]
              ↓
       (tap detectado)
              ↓
       [C-13 Confirmar con PIN]
              ↓
       [C-14 Pago exitoso]
```

**Variantes del flujo offline (cubierto en F-C-09).**

#### [C-12] Esperar tap NFC
```
┌─────────────────────────────────────┐
│ [← Cancelar]                        │
│                                     │
│      [Animación ondas NFC]         │
│                                     │
│      Acerca tu pulsera, tarjeta    │
│      o celular al POS del comercio  │
│                                     │
│      Esperando tap...                │
│                                     │
│   ┌────────────────────────┐      │
│   │ Tu saldo: Bs. 12,500.00 │      │
│   └────────────────────────┘      │
│                                     │
│   💡 Si no tienes saldo, te       │
│   avisaremos al instante.         │
│                                     │
└─────────────────────────────────────┘
```

- Animación ondas pulsantes (3-5 segundos loop)
- Si no detecta en 30s → "¿Problemas? Prueba QR"

#### [C-13] Confirmar pago con PIN
```
┌─────────────────────────────────────┐
│ [← Cancelar]                        │
│                                     │
│   Pago a Terminal La Bandera        │
│                                     │
│   Bs. 45.00                         │
│                                     │
│   Tu nuevo saldo:                  │
│   Bs. 12,455.00                     │
│                                     │
│   Ingresa tu PIN para confirmar:  │
│   ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐       │
│   │●│ │●│ │●│ │ │ │ │ │ │       │
│   └─┘ └─┘ └─┘ └─┘ └─┘ └─┘       │
│                                     │
│   ⏱️ 30 segundos para confirmar    │
│                                     │
│   [Huella]                          │
└─────────────────────────────────────┘
```

- Monto, comercio, nuevo saldo visibles
- PIN 6 dígitos
- Timer 30s, después cancela

#### [C-14] Pago exitoso
```
┌─────────────────────────────────────┐
│                                     │
│         ✓ (animado)                │
│                                     │
│      ¡Pago exitoso!                │
│                                     │
│   Bs. 45.00                         │
│   Terminal La Bandera                │
│                                     │
│   Saldo: Bs. 12,455.00             │
│                                     │
│   📋 ID: pay-uuid-corto           │
│   📅 2 oct 2026, 14:32             │
│                                     │
│   [ Ver recibo ] [ Inicio ]       │
└─────────────────────────────────────┘
```

---

## F-C-06: Pago QR (cámara)

**Pantallas:** C-15, [C-13], [C-14]

```
[Home] → [C-15 Escanear QR]
              ↓
       (QR detectado)
              ↓
       [C-13' Confirmar pago QR]
              ↓
       [C-14 Pago exitoso]
```

#### [C-15] Escanear QR del comercio
```
┌─────────────────────────────────────┐
│ [← Cancelar]                        │
│                                     │
│   ┌────────────────────────┐      │
│   │                          │      │
│   │  [Cámara con marco]    │      │
│   │                          │      │
│   │   Apunta al QR del      │      │
│   │   comercio              │      │
│   │                          │      │
│   └────────────────────────┘      │
│                                     │
│   🔲 Escaneando...                  │
│                                     │
│   💡 O usa NFC si está disponible  │
│                                     │
└─────────────────────────────────────┘
```

- Vista de cámara con marco cuadrado
- Animación de escaneo
- Si QR expira (30s) → re-scan

#### [C-13'] Confirmar pago QR
```
[Igual que C-13 pero con texto contextual del QR]
```

---

## F-C-07: Pago phone-to-phone P2P

**Pantallas:** nuevas [C-40] a [C-43]

```
[Home] → [C-40 Iniciar P2P]
            ↓
       [C-41 Buscar destinatario]
            ↓
       [C-42 Ingresar monto]
            ↓
       [C-13 Confirmar con PIN]
            ↓
       [C-43 P2P exitoso]
```

#### [C-40] Iniciar transferencia P2P
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Enviar a otra persona             │
│                                     │
│   ┌──────────────┐ ┌──────────────┐│
│   │ 📞 Por       │ │ 🔍 Por      ││
│   │ teléfono     │ │ contacto     ││
│   └──────────────┘ └──────────────┘│
│                                     │
│   ┌──────────────┐ ┌──────────────┐│
│   │ 📡 Por NFC   │ │ 📷 Por QR   ││
│   │ del otro     │ │ del otro    ││
│   │ celular      │ │ celular     ││
│   └──────────────┘ └──────────────┘│
│                                     │
└─────────────────────────────────────┘
```

- 4 modos de encontrar destinatario
- Cada uno abre flujo específico

#### [C-41] Buscar por teléfono o contacto
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Buscar destinatario                │
│                                     │
│   📞 Teléfono                      │
│   [____________________]            │
│                                     │
│   👤 Contactos                      │
│   ┌────────────────────────┐      │
│   │ María Pérez             │      │
│   │ +58 412 1112233         │      │
│   ├────────────────────────┤      │
│   │ Juan González          │      │
│   │ +58 414 5558899         │      │
│   └────────────────────────┘      │
│                                     │
│   [   Continuar   ]                │
└─────────────────────────────────────┘
```

- Búsqueda por número
- Integración con contactos del teléfono (con permiso)
- Solo muestra contactos que ya usan ViCheck

#### [C-42] Ingresar monto P2P
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   Enviar a María Pérez             │
│                                     │
│   Bs. [____________________]       │
│                                     │
│   💬 Mensaje (opcional)             │
│   [_________________________]      │
│                                     │
│   Tu saldo: Bs. 12,500.00          │
│                                     │
│   [   Continuar   ]                │
└─────────────────────────────────────┘
```

- Mensaje opcional (max 100 chars)
- Validación saldo

#### [C-43] P2P exitoso
```
┌─────────────────────────────────────┐
│                                     │
│         ✓                          │
│                                     │
│      ¡Enviaste Bs. 500!            │
│                                     │
│   A María Pérez                     │
│                                     │
│   Tu nuevo saldo: Bs. 12,000.00   │
│                                     │
│   💬 "Almuerzo de hoy"            │
│                                     │
│   [ Ver recibo ] [ Inicio ]       │
└─────────────────────────────────────┘
```

---

## F-C-08: Pago con el mismo celular (HCE)

**Variante del F-C-05 pero iniciando desde el celular del cliente.**

```
[Cliente acerca su celular al POS del comercio]
   ↓
[POS lee el celular del cliente vía NFC peer-to-peer]
   ↓
[Flujo normal C-13 → C-14]
```

**Estados especiales:**
- Pantalla del cliente debe estar encendida (no apagada)
- HCE requiere Android 7.0+

---

## F-C-09: Pago sin conexión (fallback universal)

Este flujo es **transversal** a C-05, C-06, C-07, C-08. Cuando no hay internet, todos los pagos funcionan igual, pero:

- POS y cliente firman localmente con ECDSA
- Tx queda en vault local encriptado
- Cuando vuelva internet → sync automático

**Estados offline específicos:**

#### [C-50] Banner offline persistente
```
[Top bar]
📡 Sin conexión
Algunas funciones no estarán disponibles.
```

**Funcionalidades offline:**
- ✅ Pago NFC/QR/P2P
- ✅ Ver balance (cache)
- ✅ Ver historial (cache)
- ❌ CashIn (requiere internet)
- ❌ CashOut (requiere internet)
- ❌ Emparejar tag nuevo (requiere internet primera vez)

#### [C-51] Pago offline exitoso (variante C-14)
```
┌─────────────────────────────────────┐
│                                     │
│         ✓                          │
│                                     │
│      ¡Pago exitoso!                │
│                                     │
│   Bs. 45.00                         │
│   Terminal La Bandera                │
│                                     │
│   Saldo: Bs. 12,455.00             │
│                                     │
│   📡 Se sincronizará al volver     │
│      online                        │
│                                     │
│   [ Ver recibo ] [ Inicio ]       │
└─────────────────────────────────────┘
```

- Indicador visual "📡 Se sincronizará"
- Diferencia visual con pago online (icono sutil)

#### [C-52] Indicador de sync
```
┌─────────────────────────────────────┐
│ [Top bar]                           │
│ 🔄 Sincronizando 3 tx pendientes   │
└─────────────────────────────────────┘
```

---

## F-C-10: Ver historial y detalle de transacción

**Pantallas:** C-16, C-17

**Variantes del filtro:**

```
[C-16 Historial]
   ↓
[C-16b Filtros aplicados]
   ↓
[C-17 Detalle de transacción]
```

#### [C-16] Historial completo
```
┌─────────────────────────────────────┐
│ [←]  [Filtros ▼]                  │
│                                     │
│   Hoy, 14:32                       │
│   ┌──────────────────────────┐    │
│   │ ✓ Pago La Bandera -Bs.45 │    │
│   │   🔄 Sincronizado         │    │
│   ├──────────────────────────┤    │
│   │ ✓ Recarga Banesco +Bs.5k│    │
│   │   🔄 Sincronizado         │    │
│   └──────────────────────────┘    │
│                                     │
│   Hoy, 09:15                       │
│   ┌──────────────────────────┐    │
│   │ ✓ Envío a María -Bs.500 │    │
│   │   📡 Pendiente sync       │    │
│   └──────────────────────────┘    │
│                                     │
│   Ayer                             │
│   ┌──────────────────────────┐    │
│   │ ✓ Retiro a Banesco -Bs.5k│   │
│   │   🔄 Sincronizado         │    │
│   └──────────────────────────┘    │
│                                     │
│   [Cargar más]                     │
└─────────────────────────────────────┘
```

- Agrupado por día
- Iconos por tipo (cashin, pago, P2P, cashout)
- Estado de sync visible

#### [C-16b] Filtros
```
┌─────────────────────────────────────┐
│ [✕ Cerrar]                          │
│                                     │
│   Filtrar historial                │
│                                     │
│   📅 Período                        │
│   [Hoy] [Semana] [Mes] [Año] [Custom]│
│                                     │
│   📊 Tipo                           │
│   ☑ Recargas                        │
│   ☑ Pagos                           │
│   ☑ Transferencias                  │
│   ☑ Retiros                         │
│                                     │
│   💰 Monto                          │
│   Mín [___] Máx [___]              │
│                                     │
│   [ Aplicar filtros ]              │
└─────────────────────────────────────┘
```

#### [C-17] Detalle de transacción
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│         ✓ Sincronizado              │
│                                     │
│   Bs. 45.00                         │
│                                     │
│   ━━━━━━━━━━━━━━━━━━━━━            │
│                                     │
│   Tipo:          Pago              │
│   Comercio:      Terminal La Bandera│
│   Fecha:         2 oct 2026, 14:32 │
│   ID:            pay-uuid-corto    │
│                                     │
│   ━━━━━━━━━━━━━━━━━━━━━            │
│                                     │
│   Detalles técnicos                │
│   🔐 Firma: 0x9f4a...corto         │
│   📡 Estado: Sincronizado          │
│   🔄 Sincronizado: hace 5 seg      │
│   📲 Dispositivo: Mi celular       │
│                                     │
│   [ Compartir ] [ Reportar ]       │
└─────────────────────────────────────┘
```

---

## F-C-11: Configuración de seguridad

**Pantallas:** nuevas [C-60] a [C-64]

```
[Settings → Seguridad]
   ↓
[C-60 Seguridad hub]
   ├→ [C-61 Cambiar PIN]
   ├→ [C-62 Habilitar biometría]
   ├→ [C-63 Sesiones activas]
   └→ [C-64 Bloquear app]
```

#### [C-60] Seguridad hub
```
┌─────────────────────────────────────┐
│ [←]  Seguridad                     │
│                                     │
│   🔐 PIN                            │
│   Cambiado hace 30 días  [Cambiar]│
│                                     │
│   👆 Huella digital                  │
│   Habilitado para login  [Off]     │
│                                     │
│   📱 Dispositivo de confianza        │
│   Este celular                  ✓  │
│                                     │
│   🔍 Sesiones activas               │
│   1 sesión activa  [Ver]            │
│                                     │
│   ⏱️ Bloqueo automático              │
│   [Inmediato ▼]                     │
│                                     │
└─────────────────────────────────────┘
```

---

## F-C-12: Notificaciones

**Pantallas:** nuevas [C-70] a [C-72]

```
[Top bar] → Click en push
   ↓
[C-70 Detalle de notificación]
   ↓
[Acciones: ver tx, ver perfil, etc.]
```

**Tipos:**
- ✅ Pago recibido (P2P)
- ✅ CashIn completado
- ✅ CashOut acreditado
- ✅ Tag emparejado
- ⚠️ Actividad sospechosa detectada
- 🔄 Sync completado
- 💰 Cashback recibido

---

## F-C-13: Reembolsos y disputas

**Pantallas:** nuevas [C-80] a [C-83]

```
[Detalle tx] → [Reportar problema]
   ↓
[C-80 Categoría de disputa]
   ↓
[C-81 Detalle del problema]
   ↓
[C-82 Confirmar envío]
   ↓
[C-83 Ticket creado]
```

---

## F-C-14: Recuperación de cuenta

**Pantallas:** nuevas [C-90] a [C-92]

```
[Login → "Olvidé mi PIN"]
   ↓
[C-90 Verificación identidad]
   ↓
[C-91 Reset PIN]
   ↓
[C-92 Confirmación]
```

---

## F-C-15: Eliminar cuenta

**Pantallas:** nuevas [C-100] a [C-102]

```
[Settings → Eliminar cuenta]
   ↓
[C-100 Advertencia]
   ↓
[C-101 Liquidar saldo pendiente]
   ↓
[C-102 Confirmar con PIN]
   ↓
[Cuenta eliminada en 30 días]
```

---

## F-C-16: Promoción y cashback

**Pantallas:** nuevas [C-110] a [C-112]

```
[Home → Ver promo balance]
   ↓
[C-110 Lista de promociones]
   ↓
[C-111 Detalle de promoción]
   ↓
[C-112 Reclamar cashback]
```

---

# PARTE 2 — ALIADO / COMERCIO (POS + Dashboard)

## F-A-01: Onboarding del aliado

**Pantallas:** nuevas [A-01] a [A-07]

```
[A-01 Splash POS] → [A-02 Bienvenida]
   ↓
[A-03 Registro RIF]
   ↓
[A-04 Datos del comercio]
   ↓
[A-05 Cuenta bancaria]
   ↓
[A-06 Configuración inicial]
   ↓
[A-07 Onboarding completado]
```

---

## F-A-02: Configurar branding y fees

**Pantallas:** nuevas [A-10] a [A-14]

```
[Dashboard] → [Configuración]
   ↓
[A-10 Branding]
   ├→ [A-11 Logo]
   ├→ [A-12 Colores]
   └→ [A-13 Nombre comercial]

[A-10] → [A-14 Fees y límites]
```

---

## F-A-03: Cobro NFC (POS)

**Pantallas:** P-02, P-03, P-05, P-06 + nuevo P-02b ingreso monto

```
[P-02 POS Home] → [P-02b Ingresar monto] → [P-03 Esperar tap]
                                       ↓
                                 [P-05 Procesando]
                                       ↓
                                ┌──────┴──────┐
                                ↓              ↓
                          [P-06 Éxito]   [P-07 Saldo insuficiente]
```

---

## F-A-04: Cobro QR (POS)

**Pantallas:** P-04, P-05, P-06

```
[P-02 POS Home] → [P-04 Generar QR] → (cliente escanea)
                                          ↓
                                    [P-05 Procesando]
                                          ↓
                                    [P-06 Éxito]
```

#### [P-04] POS esperando que cliente escanee
```
┌─────────────────────────────────────┐
│ [← Cancelar]                        │
│                                     │
│   Monto: Bs. 45.00                  │
│                                     │
│   ┌────────────────────────┐      │
│   │                          │      │
│   │   [QR dinámico]         │      │
│   │   Se renueva en 25s    │      │
│   │                          │      │
│   └────────────────────────┘      │
│                                     │
│   El cliente debe escanear el QR   │
│   desde su app ViCheck            │
│                                     │
│   🔄 QR válido por 30 segundos    │
│                                     │
└─────────────────────────────────────┘
```

---

## F-A-05: Cobro manual (POS)

**Pantallas:** nuevas [P-12] a [P-14]

```
[P-02 POS Home] → [P-12 Cobro manual]
   ↓
[P-13 Ingresar teléfono del cliente]
   ↓
[P-14 Ingresar monto + descripción]
   ↓
[Push al cliente para confirmar]
   ↓
[Éxito / Cancelado]
```

---

## F-A-06: Sync de transacciones (POS)

**Pantallas:** P-09 + nuevas [P-15] a [P-18]

```
[P-02] → Indicador "5 pendientes"
   ↓
[P-15 Sync ahora] (manual)
   ↓
[P-16 Progreso de sync]
   ↓
[P-17 Resultado del sync]
   ↓
[P-18 Detalle de tx fallida]
```

---

## F-A-07: CashOut / Liquidación a banco del aliado

**Pantallas:** P-11 + nuevas [P-20] a [P-23]

```
[P-11 POS CashOut]
   ↓
[P-20 Seleccionar monto]
   ↓
[P-21 Confirmar]
   ↓
[P-22 Procesando]
   ↓
[P-23 Liquidación iniciada]
```

---

## F-A-08: Dashboard de ventas (móvil gerente)

**Pantallas:** nuevas [D-01] a [D-08]

```
[D-01 Dashboard home]
   ├→ [D-02 Ventas hoy]
   ├→ [D-03 Ventas semana]
   ├→ [D-04 Ventas mes]
   ├→ [D-05 Comparativa]
   ├→ [D-06 Productos top]
   ├→ [D-07 Por empleado]
   └→ [D-08 Gráficos]
```

#### [D-01] Dashboard home del aliado
```
┌─────────────────────────────────────┐
│ Terminal La Bandera    [usuario ▼]  │
│                                     │
│  ┌────────────────────────────┐    │
│  │ 📊 Ventas hoy                │    │
│  │ Bs. 12,450.00                │    │
│  │ 87 transacciones            │    │
│  │ ↗ +15% vs ayer              │    │
│  └────────────────────────────┘    │
│                                     │
│  ┌──────────────┐ ┌──────────────┐ │
│  │ 💰 Saldo     │ │ 📡 Sync      │ │
│  │ Bs. 8,500   │ │ 0 pendientes │ │
│  └──────────────┘ └──────────────┘ │
│                                     │
│  Top productos                      │
│  1. Pasaje Bs.500    320 ventas │
│  2. Trato Total Bs.800  90 ventas │
│  3. VIP Bs.1500       45 ventas │
│                                     │
│  [Cobrar] ahora [Más...]           │
└─────────────────────────────────────┘
```

---

## F-A-09: Gestión de empleados

**Pantallas:** nuevas [D-10] a [D-15]

```
[D-01] → [D-10 Empleados]
   ├→ [D-11 Agregar empleado]
   ├→ [D-12 Permisos por empleado]
   ├→ [D-13 Horarios]
   └→ [D-14 Eliminar empleado]
```

---

## F-A-10: Notificación de cashin recibido del cliente

**Pantalla:** nueva [D-20]

Cuando un cliente recarga saldo desde su banco, **el aliado recibe una notificación push** si tiene configurado el feature "Notificarme cuando mis clientes recargan":

```
┌─────────────────────────────────────┐
│ [Push notification]                │
│                                     │
│ 💰 Juan Pérez recargó Bs. 2,000   │
│ Toca para ver detalles             │
└─────────────────────────────────────┘
```

#### [D-20] Detalle de notificación
```
┌─────────────────────────────────────┐
│ [←]                                │
│                                     │
│   💰 Recarga de cliente             │
│                                     │
│   Juan Pérez                         │
│   +58 412 1234567                   │
│                                     │
│   Monto: Bs. 2,000.00              │
│                                     │
│   Fecha: 2 oct 2026, 14:32         │
│                                     │
│   💡 Notificación informativa.     │
│   No afecta tu saldo directamente.│
│                                     │
└─────────────────────────────────────┘
```

**Configuración:** Settings → Notificaciones → Toggle "Cashins de clientes"

---

## F-A-11: Gestión de promociones

**Pantallas:** nuevas [D-30] a [D-35]

```
[D-01] → [D-30 Promociones]
   ├→ [D-31 Crear promo]
   ├→ [D-32 Cashback]
   ├→ [D-33 Descuento]
   └→ [D-34 Activas]
```

---

# PARTE 3 — WEB ADMIN VIPPO

## F-W-01: Login admin + 2FA

**Pantallas:** nuevas [W-01] a [W-03]

```
[W-01 Login]
   ↓
[W-02 2FA TOTP]
   ↓
[W-03 Dashboard]
```

#### [W-01] Login admin
```
┌──────────────────────────────────────────┐
│                                          │
│         [Logo ViCheck Admin]              │
│                                          │
│   Iniciar sesión                         │
│                                          │
│   Email                                   │
│   [admin@vippo.io____________]            │
│                                          │
│   Contraseña                              │
│   [_______________]                      │
│                                          │
│   [   Iniciar sesión   ]                 │
│                                          │
│   ¿Olvidaste tu contraseña?             │
│                                          │
└──────────────────────────────────────────┘
```

#### [W-02] 2FA TOTP
```
┌──────────────────────────────────────────┐
│                                          │
│   Verificación en dos pasos             │
│                                          │
│   Ingresa el código de tu app          │
│   autenticadora (Google Authenticator) │
│                                          │
│   ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐              │
│   │ │ │ │ │ │ │ │ │ │ │ │              │
│   └─┘ └─┘ └─┘ └─┘ └─┘ └─┘              │
│                                          │
│   💡 El código cambia cada 30 segundos │
│                                          │
│   [   Verificar   ]                     │
│                                          │
└──────────────────────────────────────────┘
```

---

## F-W-02: Ver lista de aliados (tenants)

**Pantalla:** [W-10] Lista de tenants

```
┌──────────────────────────────────────────────────────────────┐
│ ViPPO Admin        Dashboard | Aliados | Usuarios | Seguridad│
├──────────────────────────────────────────────────────────────┤
│ Aliados                                          [+ Crear]    │
│                                                              │
│ 🔍 [Buscar aliado...]      Estado: [Todos ▼] [Activos ▼]  │
│                                                              │
│ ┌──────────────┬─────────┬──────────┬──────────┬─────────┐ │
│ │ Nombre       │ RIF     │ Estado   │ Balance  │ Ventas  │ │
│ ├──────────────┼─────────┼──────────┼──────────┼─────────┤ │
│ │ La Bandera   │ J-1234  │ Activo   │ Bs.8,500 │ Bs.45k  │ │
│ │ Comida Rapida│ J-5678  │ Activo   │ Bs.2,300 │ Bs.22k  │ │
│ │ VIP Disco    │ VIP-001  │ Suspend. │ Bs.0     │ Bs.0    │ │
│ └──────────────┴─────────┴──────────┴──────────┴─────────┘ │
│                                                              │
│ Página 1 de 23                       [Anterior] [Siguiente]  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## F-W-03: Crear/editar/deshabilitar aliado

**Pantallas:** nuevas [W-20] a [W-23]

```
[W-10] → [+ Crear]
   ↓
[W-20 Datos básicos]
   ↓
[W-21 Cuenta bancaria]
   ↓
[W-22 Configuración inicial]
   ↓
[W-23 Confirmar y crear]

[Editar: misma W-20 pero precargada]
[Deshabilitar: W-24 confirmación]
```

#### [W-20] Datos básicos del aliado
```
┌──────────────────────────────────────────────────────────────┐
│ Crear aliado                                  [Cancelar]      │
│                                                              │
│ Datos básicos                                                │
│                                                              │
│ Razón social *                                                 │
│ [Terminal La Bandera C.A.____________]                       │
│                                                              │
│ RIF *                                                        │
│ [J-12345678-9_____]                                          │
│                                                              │
│ Nombre comercial *                                           │
│ [Terminal La Bandera________]                                │
│                                                              │
│ Tipo de negocio *                                            │
│ [Transporte ▼]                                              │
│                                                              │
│ Teléfono de contacto *                                       │
│ [+58 ____________________]                                  │
│                                                              │
│ Email                                                        │
│ [_________________________]                                  │
│                                                              │
│                              [Siguiente →]                  │
└──────────────────────────────────────────────────────────────┘
```

---

## F-W-04: Ver lista de usuarios

**Pantalla:** [W-30]

```
┌──────────────────────────────────────────────────────────────┐
│ Usuarios                                                     │
│                                                              │
│ 🔍 [Buscar...]  Estado: [Todos ▼]  Tenant: [Todos ▼]      │
│                                                              │
│ ┌──────────────┬──────────────┬──────────┬──────────┬─────┐ │
│ │ Nombre       │ Cédula       │ Tenant   │ Balance  │ KYC │ │
│ ├──────────────┼──────────────┼──────────┼──────────┼─────┤ │
│ │ Juan Pérez   │ V-12345678   │ La Band. │ Bs.12.5k │Full │ │
│ │ María López  │ V-87654321   │ VIP Disco│ Bs.2,300 │Bas  │ │
│ └──────────────┴──────────────┴──────────┴──────────┴─────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## F-W-05: CRUD de datos de usuario

**Pantallas:** nuevas [W-40] a [W-45]

```
[W-30] → Click en usuario
   ↓
[W-40 Detalle de usuario]
   ├→ [W-41 Editar datos]
   ├→ [W-42 Cambiar estado]
   ├→ [W-43 Ver wallets]
   ├→ [W-44 Ver transacciones]
   └→ [W-45 Eliminar usuario]
```

#### [W-40] Detalle de usuario
```
┌──────────────────────────────────────────────────────────────┐
│ [← Usuarios]                              [Editar] [Deshabilitar]│
│                                                              │
│ Juan Pérez                                                   │
│ V-12345678                                                   │
│                                                              │
│ ┌─────────────────────┬────────────────────┐               │
│ │ Datos personales     │ Cuentas            │               │
│ │ Nombre: Juan Pérez   │ Tenant: La Bandera │               │
│ │ Cédula: V-12345678   │ Wallet ID:        │               │
│ │ Email: juan@x.com    │ Balance: Bs.12.5k │               │
│ │ Tel: +58 412 1234567 │ Status: Active     │               │
│ │ KYC: Full            │                    │               │
│ └─────────────────────┴────────────────────┘               │
│                                                              │
│ KYC Status                                                   │
│ ┌────────────────────────────────────────────────┐         │
│ │ Documento identidad: ✓ Verificado              │         │
│ │ Selfie: ✓ Verificado                          │         │
│ │ Prueba de vida: ✓ Verificado                  │         │
│ │ Listas restrictivas: ✓ Limpio                  │         │
│ │ PEP: No es PEP                                │         │
│ └────────────────────────────────────────────────┘         │
│                                                              │
│ Actividad reciente                                           │
│ ┌────────────────────────────────────────────────┐         │
│ │ Hoy 14:32 - Pago La Bandera -Bs.45            │         │
│ │ Hoy 09:15 - Recarga Banesco +Bs.5,000         │         │
│ │ Ayer 18:00 - Retiro a Banesco -Bs.2,000       │         │
│ └────────────────────────────────────────────────┘         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## F-W-06: Verificaciones de seguridad y KYC

**Pantallas:** nuevas [W-50] a [W-55]

```
[W-40] → [Verificaciones]
   ↓
[W-50 Lista de verificaciones pendientes]
   ↓
[W-51 Revisar documento de identidad]
   ↓
[W-52 Revisar selfie]
   ↓
[W-53 Verificación de vida]
   ↓
[W-54 Aprobar / Rechazar]
   ↓
[W-55 Confirmación]
```

#### [W-50] Lista de verificaciones pendientes
```
┌──────────────────────────────────────────────────────────────┐
│ Verificaciones de seguridad pendientes           [Filtrar ▼]│
│                                                              │
│ ┌─────────────┬────────────┬──────────────┬────────────┐   │
│ │ Usuario     │ Tipo       │ Recibido     │ Acción      │   │
│ ├─────────────┼────────────┼──────────────┼────────────┤   │
│ │ Juan Pérez  │ Cédula     │ Hace 2h     │ [Revisar]  │   │
│ │ María López │ Selfie      │ Hace 5h     │ [Revisar]  │   │
│ │ Carlos R.   │ Prueba vida│ Hace 1d     │ [Revisar]  │   │
│ └─────────────┴────────────┴──────────────┴────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

#### [W-51] Revisar documento
```
┌──────────────────────────────────────────────────────────────┐
│ [←]                                                      [Cerrar]│
│                                                              │
│ Verificación de cédula - Juan Pérez                          │
│                                                              │
│ ┌────────────────────┐  ┌────────────────────┐               │
│ │  [Foto cédula]     │  │ [Selfie]           │               │
│ │  Frente            │  │                    │               │
│ └────────────────────┘  └────────────────────┘               │
│                                                              │
│ ┌────────────────────┐                                       │
│ │  [Foto cédula]     │                                       │
│ │  Reverso           │                                       │
│ └────────────────────┘                                       │
│                                                              │
│ Análisis automático:                                         │
│ • Documento válido: ✓                                        │
│ • Cédula coincide: ✓                                         │
│ • Selfie coincide: ✓                                         │
│ • No manipulación detectada: ✓                               │
│                                                              │
│ ⚠️ Documento tiene 8 años (vigencia 10 años)              │
│                                                              │
│ Aprobado por: [yo ▼]    Fecha: [hoy]                       │
│                                                              │
│ Decisión:                                                    │
│ ( ) Aprobar                                                   │
│ ( ) Rechazar - Razón: [________________]                    │
│ ( ) Solicitar más info                                       │
│                                                              │
│ [   Confirmar decisión   ]                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## F-W-07: Anti-fraude: revisar tx sospechosas

**Pantallas:** nuevas [W-60] a [W-65]

```
[W-Dashboard] → [Anti-fraude]
   ↓
[W-60 Cola de alertas]
   ↓
[W-61 Detalle de alerta]
   ↓
[W-62 Acciones posibles]
```

#### [W-60] Cola de alertas anti-fraude
```
┌──────────────────────────────────────────────────────────────┐
│ Anti-fraude - Alertas                                       │
│                                                              │
│ Filtros: [Severidad ▼] [Tipo ▼] [Tenant ▼]                 │
│                                                              │
│ 🔴 3 críticas | 🟡 12 advertencia | 🔵 8 informativa      │
│                                                              │
│ ┌─────────┬─────────┬──────────┬──────────┬──────────────┐ │
│ │ Severidad│ Tipo    │ Usuario  │ Tenant   │ Detectado    │ │
│ ├─────────┼─────────┼──────────┼──────────┼──────────────┤ │
│ │ 🔴      │Tx rapid │ Juan P.  │La Bandera│ Hace 5 min   │ │
│ │ 🟡      │Monto    │ María L. │VIP Disco │ Hace 1h      │ │
│ │ 🔵      │Geo      │ Carlos R.│Comida Rap│ Hace 3h      │ │
│ └─────────┴─────────┴──────────┴──────────┴──────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

#### [W-61] Detalle de alerta
```
┌──────────────────────────────────────────────────────────────┐
│ [←]  Alerta: Tx rápidas (🔴 crítica)                        │
│                                                              │
│ Juan Pérez (+58 412 1234567)                                │
│ La Bandera                                                    │
│                                                              │
│ Actividad sospechosa:                                        │
│ • 15 transacciones en 2 minutos                             │
│ • Promedio: Bs. 200                                          │
│ • Total: Bs. 3,000                                          │
│                                                              │
│ Patrón normal del usuario:                                   │
│ • 2 transacciones/día                                        │
│ • Promedio: Bs. 50                                            │
│                                                              │
│ Análisis:                                                    │
│ 🔴 Velocidad 7.5x sobre lo normal                           │
│ 🔴 Monto total 60x sobre promedio                           │
│ 🟡 IP cambió de Maracaibo a Caracas en 10 min                │
│                                                              │
│ Acciones disponibles:                                        │
│ [🟡 Marcar tx sospechoso]                                    │
│ [❄️ Congelar wallet]                                       │
│ [🔄 Reversar transacción]                                  │
│ [✅ Descartar alerta (falso positivo)]                      │
│                                                              │
│ Notas del analista:                                          │
│ [____________________________________________________]     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## F-W-08: Anti-fraude: congelar wallet

**Pantalla:** [W-70]

```
┌──────────────────────────────────────────────────────────────┐
│ ❄️ Congelar wallet                                          │
│                                                              │
│ Wallet: Juan Pérez (w-uuid-corto)                          │
│ Balance actual: Bs. 12,500.00                               │
│                                                              │
│ Razón *                                                      │
│ [Actividad fraudulenta detectada____________________]       │
│                                                              │
│ Tipo de congelamiento:                                       │
│ ( ) Temporal (24h, requiere renovación)                     │
│ (●) Indefinido (hasta revisión manual)                      │
│                                                              │
│ ⓘ Esta acción:                                              │
│ • Impide nuevas operaciones                                  │
│ • Notifica al usuario vía push                               │
│ • Se registra en audit log                                   │
│                                                              │
│ [   Confirmar congelamiento   ]      [Cancelar]             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## F-W-09: Anti-fraude: reversar transacción

**Pantalla:** [W-80]

```
┌──────────────────────────────────────────────────────────────┐
│ 🔄 Reversar transacción                                     │
│                                                              │
│ TX original: pay-uuid-corto                                  │
│ Juan Pérez → La Bandera                                      │
│ Bs. 45.00                                                   │
│ 2 oct 2026, 14:32                                          │
│                                                              │
│ ⚠️ Esta acción:                                              │
│ • Devuelve el monto al cliente                              │
│ • Descuenta al comercio                                      │
│ • Crea transacción de reverso en ledger                      │
│ • Se registra en audit log                                   │
│ • No se puede deshacer                                      │
│                                                              │
│ Razón *                                                      │
│ [Actividad fraudulenta confirmada_______________________]   │
│                                                              │
│ Confirmar con credencial de admin:                          │
│ [contraseña admin]                                          │
│                                                              │
│ [   Reversar transacción   ]      [Cancelar]                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## F-W-10: Conciliación manual

**Pantallas:** nuevas [W-90] a [W-92]

```
[Dashboard] → [Conciliación]
   ↓
[W-90 Vista general]
   ↓
[W-91 Detalle de discrepancia]
   ↓
[W-92 Acciones correctivas]
```

---

## F-W-11: Reportes regulatorios (SENIAT)

**Pantallas:** nuevas [W-100] a [W-105]

```
[Reportes] → [Regulatorios]
   ↓
[W-100 Seleccionar tipo de reporte]
   ├→ [W-101 Reporte SENIAT]
   ├→ [W-102 Reporte SENIAT]
   └→ [W-103 Auditoría LOPDP]
```

---

## F-W-12: Configuración global de fees y límites

**Pantallas:** nuevas [W-110] a [W-115]

```
[Configuración] → [Global]
   ↓
[W-110 Fees]
   ├→ [W-111 Fee por defecto]
   ├→ [W-112 Fee por tipo de tx]
   └→ [W-113 Fee por tenant]

[W-110] → [W-114 Límites]
   ├→ [W-115 Límites CashIn]
   └→ [W-116 Límites CashOut]
```

---

# PARTE 4 — DIAGRAMAS DE NAVEGACIÓN

## Mapa de navegación cliente (Android)

```
                    [Splash]
                       ↓
                [Onboarding]
                       ↓
            ┌─────────┴─────────┐
            ↓                   ↓
         [Login]         [Registro]
            ↓                   ↓
            └─────────┬─────────┘
                      ↓
              [Verificación OTP]
                      ↓
            ┌─────────┴─────────┐
            ↓                   ↓
      [Datos pers.]    [Selección banco]
            ↓                   ↓
            └─────────┬─────────┘
                      ↓
              [Home / Billetera]
                   │  │  │
       ┌───────────┼──┼───────────┐
       ↓           ↓  ↓           ↓
   [CashIn]   [CashOut] [Pagar] [Historial]
       │           │      │          │
       │           │   ┌──┼──┐       │
       │           │   ↓   ↓       ↓
       │           │ [NFC] [QR] [Detalle tx]
       │           │   │   │
       │           │   └───┘
       │           ↓     ↓
       │       [PIN] [Procesando]
       │           │   │
       │           ↓   ↓
       │       [Éxito / Fallo]
       │           │
       │           └──→ [Home]
       │
       └──→ [Home]
              ↑
       [Settings] ←─── [Emparejar tag]
              ↑
       [Seguridad]
```

## Mapa de navegación aliado

```
                [Splash POS]
                     ↓
                [Login POS]
                     ↓
              ┌──────┴──────┐
              ↓             ↓
       [POS Home]    [Dashboard Aliado]
              │             │
       ┌──────┼─────┐       │   │
       ↓      ↓     ↓       ↓   ↓
   [Cobrar NFC] [QR] [Manual] [Empleados] [Reportes]
       │      │     │       │
       └──────┴─────┘       │
              ↓              │
        [Esperando tap/QR]   │
              ↓              │
        [Procesando]         │
              ↓              │
       ┌──────┴─────┐        │
       ↓            ↓        ↓
    [Éxito]    [Fallo]   [Liquidación]
       │            │        │
       └──────┬─────┘        │
              ↓          ↓
          [POS Home] ←─────┘
```

## Mapa de navegación web admin

```
            [Login Admin]
                  ↓
           [2FA TOTP]
                  ↓
           [Dashboard]
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
   [Aliados] [Usuarios] [Seguridad]
       │          │          │
       │          │     ┌────┴────┐
       │          │     ↓         ↓
       │          │  [Anti-fraude] [KYC]
       │          │     │         │
       │          │     ↓         ↓
       │          │  [Alertas] [Verificaciones]
       │          │     │         │
       │          │     └───┬─────┘
       │          │         ↓
       │          │    [Detalle → Acciones]
       │          │         ↓
       │          │    [Congelar / Reversar]
       │          │
       │          └→ [CRUD]
       │              ↓
       │          [Editar / Eliminar]
       │
       └→ [CRUD aliado]
              ↓
          [Detalle / Crear / Editar]
              ↓
          [Branding / Fees / Cuenta banco]
```

---

# PARTE 5 — ESTADOS GLOBALES

## Estados de la app cliente

| Estado | Indicador visual | Funcionalidades afectadas |
|---|---|---|
| `Online` | Barra verde sutil "🟢 Online" | Todas disponibles |
| `Offline` | Barra naranja "📡 Sin conexión" | CashIn/Out deshabilitados |
| `Sincronizando` | "🔄 Sincronizando X tx" | Resto funciona normal |
| `Error de sync` | "⚠️ Error sincronizando" | Acción manual requerida |
| `Batería baja` | Banner amarillo | Recomienda cargar pronto |
| `Modo avión` | Banner persistente | Todas las NFC deshabilitadas |

## Estados de error globales

| Código | Mensaje al usuario | Acción |
|---|---|---|
| `ERR_NETWORK` | "Sin conexión. Verifica tu internet" | Mostrar modo offline |
| `ERR_AUTH_EXPIRED` | "Tu sesión expiró. Inicia sesión" | Redirect a login |
| `ERR_SERVER` | "Error del servidor. Intenta más tarde" | Retry con backoff |
| `ERR_VALIDATION` | Específico del campo | Highlight del input |
| `ERR_INSUFFICIENT_FUNDS` | "Saldo insuficiente" | Ofrecer CashIn |
| `ERR_TAG_NOT_REGISTERED` | "Tu tag no está registrado" | Flujo emparejar |
| `ERR_TAG_BLOCKED` | "Tu tag fue bloqueado. Llama soporte" | Mostrar teléfono soporte |
| `ERR_RATE_LIMIT` | "Demasiadas operaciones. Espera N min" | Countdown |
| `ERR_WALLET_BLOCKED` | "Tu cuenta está bloqueada" | Solo logout + soporte |

---

# PARTE 6 — CONSIDERACIONES TRANSVERSALES

## Multi-tenancy visual

**Cada aliado puede customizar:**
- Logo en pantallas del cliente (aparece en收款/pago)
- Color primario (botones, headers)
- Mensajes personalizados ("Gracias por tu compra en La Bandera")

**Defaults para clientes:**
- Si el aliado no customizó → tema estándar ViCheck
- Cache de branding por tenant

## Accesibilidad

- Touch targets ≥ 48dp en todos los botones
- Contraste WCAG AA mínimo
- Soporte screen readers (TalkBack)
- Texto escalable (dynamic type)
- Navegación por gestos opcional
- Modo de alto contraste

## Modo oscuro

- Mismo sistema de tokens pero con paleta invertida
- Primary 100 como background principal
- Primary 900 como texto

## i18n (futuro)

- Español (default para VE)
- Inglés (internacional)
- Textos parametrizados, no hardcoded

---

# PARTE 7 — MÉTRICAS DE UX

| Métrica | Target | Cómo medir |
|---|---|---|
| Tiempo promedio de registro | < 2 min | Analytics |
| Tiempo de CashIn completo | < 30s | Analytics |
| Tiempo de pago NFC completo | < 5s (3 taps) | Analytics |
| Tasa de éxito de emparejar tag | > 95% | Analytics |
| Tasa de abandono en PIN | < 10% | Analytics |
| Crash-free users | > 99% | Crashlytics |
| NPS cliente final | > 50 | Survey |
| NPS aliado | > 40 | Survey |

---

# PARTE 8 — ENTREGABLES PARA FIGMA

## Prioridad para Figma (orden recomendado)

1. **Sistema de diseño base** (paleta, tipografía, componentes)
2. **Flujos de cliente (F-C-01 → F-C-09)** — los más críticos
3. **Flujos de POS (F-A-03 → F-A-06)** — el producto base
4. **Dashboard aliado (F-A-08)** — diferenciador
5. **Web admin (F-W-01 → F-W-06)** — para gestión
6. **Anti-fraude (F-W-07 → F-W-09)** — seguridad
7. Resto de flujos

## Componentes críticos para entregar primero

- Botones (primary, secondary, tertiary, destructive)
- Inputs (text, phone, currency, PIN, OTP)
- Cards (balance, transacción, info, error)
- Modales y bottom sheets
- Indicadores de estado (online/offline, sync, loading)
- Empty states y error states
- Notificaciones push
- Componentes NFC/QR
- Componentes de seguridad (PIN, biometría)

---

**Documento completo. Total: ~60 pantallas únicas + 30+ flujos. Mantenido con la estructura de figma-base-spec.md.**

**Próximo documento:** Cuando tengas feedback de este, puedo crear diagramas Mermaid de los flujos o expandir el detalle de pantallas individuales.