# PPG Wave Clone — Fase 7: Interfaz Profesional

Implementación original de un instrumento virtual VST3 de síntesis wavetable
inspirado conceptualmente en el PPG Wave 3.3. No contiene ROMs, muestras,
gráficos ni código propietario de terceros.

## Estado de esta entrega

**Fases 1-6**: listo y confirmado sonando en hardware real (arquitectura,
osciladores wavetable, filtro + envolventes, LFOs + matriz de modulación,
efectos, gestión de presets).

**Fase 7** (esta entrega): interfaz profesional definitiva.

- `PPGLookAndFeel`: estética propia de instrumento digital de los años 80
  — paneles oscuros, knobs con arco de valor (no el rotary plano genérico
  de JUCE), acentos cian/ámbar. Interpretación original, sin logotipos ni
  gráficos de terceros. Se aplica una sola vez (`setLookAndFeel`), así que
  todos los controles existentes se redibujan solos.
- Cada sección (osciladores, filtro, envolventes, LFOs, matriz, efectos)
  ahora vive dentro de un panel con fondo y borde propios — el agrupamiento
  visual de "módulos" que pedía el documento de diseño.
- **Visualización gráfica ADSR** en los dos paneles de envolvente
  (`EnvelopeVisualizer`): dibuja la forma attack/decay/sustain/release en
  vivo, actualizándose mientras mueves los knobs.
- **Indicador de actividad MIDI**: un LED que se enciende brevemente cada
  vez que llega MIDI al plugin.
- Ventana bastante más compacta (640×770, antes 900×1100) — misma
  cantidad de controles, con más densidad.

**Lo que NO incluye todavía**: Vintage Character (Fase 8, pendiente).

## Problemas conocidos

- **Sin indicador de CPU**: necesitaría infraestructura de medición del
  tiempo de audio callback que no existe todavía. No es imprescindible
  para que la interfaz se vea y sienta profesional, así que se deja para
  cuando haga falta específicamente.
- **Sin buscador de texto ni filtro de categoría en vivo para presets**:
  `PresetManager::findMatching()` ya los soporta a nivel de datos desde la
  Fase 6; conectarlos a controles reales es una extensión directa de esta
  misma pantalla, no un cambio de arquitectura.
- **Sync/Retrigger de LFO y Sync/Sync Division de Delay siguen sin knob
  dedicado** (mismo criterio que fases anteriores — automatizables desde
  Ableton Live).
- Sigue el mismo aliasing en notas agudas de fases anteriores
  (comportamiento esperado, no un bug).

## Compilación sin instalar nada (recomendado)

Este proyecto incluye `.github/workflows/build.yml`: compila el VST3 en un
Mac con Xcode ya instalado, en la nube, gratis, vía GitHub Actions. Sube la
carpeta a un repositorio (con GitHub Desktop, para no perder `.github` por
ser una carpeta oculta), espera la palomita verde en la pestaña "Actions", y
descarga el `.vst3` desde "Artifacts".

## Compilación local (requiere CMake + compilador C++20)

1. Clona JUCE dentro de esta carpeta:
   ```
   git clone --depth 1 --branch 7.0.12 https://github.com/juce-framework/JUCE.git
   ```
   Debe quedar como `PPGWaveClone/JUCE/`.

2. Configura y compila:

   **macOS** (Xcode instalado):
   ```
   cmake -B build -G Xcode
   cmake --build build --config Release
   ```
   El `.vst3` queda en `build/PPGWaveClone_artefacts/Release/VST3/`.
   Cópialo a `~/Library/Audio/Plug-Ins/VST3/` (tu usuario) o
   `/Library/Audio/Plug-Ins/VST3/` (todo el sistema — recomendado si Ableton
   corre en otra sesión de usuario). Si lo bajaste de un navegador:
   ```
   xattr -dr com.apple.quarantine "/ruta/al/PPG Wave Clone.vst3"
   ```

   **Windows** (Visual Studio 2022):
   ```
   cmake -B build -G "Visual Studio 17 2022"
   cmake --build build --config Release
   ```
   El `.vst3` queda en `build/PPGWaveClone_artefacts/Release/VST3/`.
   Cópialo a `C:\Program Files\Common Files\VST3\`.

3. Rescanea plugins en Ableton Live (Preferences → Plug-ins → Rescan).

## Estructura

```
PPGWaveClone/
  CMakeLists.txt
  .github/workflows/build.yml   — compila el VST3 en la nube (macOS, x86_64)
  Source/
    PluginProcessor.h/.cpp      — AudioProcessor, APVTS, Synthesiser, efectos, presets, actividad MIDI
    PluginEditor.h/.cpp         — GUI (Fase 7): paneles con estilo propio
    UI/
      PPGLookAndFeel.h/.cpp       — estética propia (knobs, combos, botones)
      StatusLED.h                 — LED reutilizable (actividad MIDI)
      EnvelopeVisualizer.h/.cpp   — forma ADSR en vivo
    Params/  ...                  (sin cambios de fondo esta fase)
    DSP/     ...                  (sin cambios de fondo esta fase)
    Synth/   ...                  (sin cambios de fondo esta fase)
    Presets/ ...                  (sin cambios de fondo esta fase)
```

## Próxima fase (Fase 8 — Vintage Character, ahora sí)

- `Source/DSP/VintageCharacter.h/.cpp`: bit depth, reducción de sample
  rate, ruido digital, drift de pitch, variación de filtro/voz por nota.
- `Oscillator::setAliasingAmount()`: reducción deliberada de resolución de
  lectura de la wavetable (aliasing como carácter, no como límite técnico).
- Control global Vintage/Modern que escala todo lo anterior.
- Integración en `SynthVoice` (instancia por voz, para variación real
  entre voces) y un panel más en la GUI, siguiendo el mismo patrón visual
  que el resto.

Dime cuándo avanzamos a la Fase 8.
