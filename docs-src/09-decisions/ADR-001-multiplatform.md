# ADR-001: Kotlin Multiplatform para shared core

**Fecha:** 2026-10-02
**Estado:** Propuesto
**Autor:** Francisco August

## Contexto

ViCheck MVP requiere app móvil con lógica compleja offline-first (vault local, ECDSA firmas, sync engine). MVP es solo Android por restricción de recursos, pero existe probabilidad alta de requerir iOS en el futuro cercano.

## Decisión

Adoptar **Kotlin Multiplatform (KMP)** desde el día 1, con:

- Shared core (lógica de negocio, crypto, sync) en KMP
- UI Android en Jetpack Compose
- Shared core code reutilizable si en el futuro se agrega iOS

## Opciones consideradas

### Opción A: Native Android solo (Kotlin)
- ✅ Setup simple, build rápido
- ✅ Toda la documentación aplica
- ❌ Si se quiere iOS en el futuro, hay que reescribir todo

### Opción B: Flutter
- ✅ Misma UI en iOS/Android
- ❌ Lenguaje Dart (no Kotlin)
- ❌ Equipo no tiene experiencia en Dart
- ❌ Performance crypto: FFI overhead

### Opción C: React Native
- ✅ JavaScript ecosystem
- ❌ Bridge JS-Native agrega latencia
- ❌ Crypto de baja latencia es problemático

### Opción D: Kotlin Multiplatform ✅ (elegida)
- ✅ Shared core reutilizable
- ✅ Kotlin es skill del equipo
- ✅ Crypto performante (nativo)
- ✅ Compose Multiplatform maduro
- ⚠️ Curva de aprendizaje inicial
- ⚠️ Algunas APIs Android requieren expect/actual

## Consecuencias

**Positivas:**
- Si se requiere iOS, no se duplica código
- Tests del shared core corren en una sola plataforma
- Lógica de vault / sync en una sola implementación
- Equipo mantiene skills Kotlin existentes

**Negativas:**
- ~3-5 días extra de setup inicial vs native Android
- Algunos patrones Android requieren expect/actual (NFC Foreground Dispatch, WiFi Direct)
- Build times ligeramente mayores
- Documentación y comunidad un poco menos madura que native Android

## Plan de mitigación

1. Empezar con POC de KMP semana 1 (solo "Hola mundo" + shared lib)
2. Validar que Foreground Dispatch funciona con expect/actual
3. Si hay bloqueos graves en semana 1, fallback a Opción A

## Referencias

- https://kotlinlang.org/docs/multiplatform.html
- https://www.jetbrains.com/compose-multiplatform/
- Apps en producción con KMP: Cash App, Block, Mercado Pago, Philips, Vinted