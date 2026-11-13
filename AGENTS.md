# Reglas del Repositorio Explainer (`AGENTS.md`)

Estas reglas aplican a cualquier modificación sobre el motor, el compilador declarativo (`explainer.json` -> `index.html`), las escenas en Canvas o los scripts de audio y renderizado.

## 1. Invariante de Subtítulos en Render (`?render=1` vs `?bare=1`)
- La clase `.clean` (activada con `?render=1` o Modo grabación) oculta únicamente controles interactivos (`.topbar`, `.pz`, `.replay`) y **nunca** debe ocultar `.subs`.
- Solo la clase `.bare` (`?bare=1` o atajo `S`) oculta `.subs` para exportaciones limpias sin texto.
- Conservá `.frame.frozen .subs, .frame.frozen .subs .w { animation: none !important; transition: none !important; }` para que Playwright capture los subtítulos con opacidad completa en cada cuadro, sin quedar a mitad de una transición CSS.

## 2. Invariante de Sincronía `audio_dur` y Anclas `F.at()`
- Al sintetizar voz por escena o generar `AUDIO_META`, almacená siempre `audio_dur` (duración hablada real sin pausa final de búfer) además de `start` y `dur`.
- Antes de construir `chunks(s.words)` y `makeAt(s)`, escalá proporcionalmente los tiempos estimados `s.words` respecto a `realSpeech = audio_dur` (`scale = realSpeech / estSpeech`) para que los bloques de subtítulos y el resaltado `karaoke` concluyan al terminar la locución, nunca durante el silencio intermedio.
- `makeAt(s)` debe emitir `console.warn` cuando una palabra ancla no aparezca en `SCRIPT[i]`, de modo que `scripts/simular.cjs` detecte y falle ante anclas desincronizadas.

## 3. Determinismo y Composición Adaptativa en Canvas (`draw(t, F)`)
- Todo cuadro debe ser función pura del tiempo global `T` / local `t`, `F.g` y el generador con semilla `rng(seed)`. Prohibido usar `Date.now()`, `performance.now()` o `Math.random()` dentro de `draw(t, F)`.
- Calculá coordenadas y tamaños desde `F.area` (`A.x0`, `A.cx`, `A.cy`, `A.s`) y tokens CSS (`C`, `FONT`). Nunca uses píxeles fijos pensados solo para `9:16`.
- Prohibido usar emojis Unicode dentro de `txt(...)` o rótulos de interfaz que repitan la locución; dibujá íconos vectoriales del motor (`ring`, `brackets`, `arrow`, `check`, `cross`, `person`, `bubble`, `doc`, `laptop`, `terminal`, `magnifier`, `padlock`, `calendar`, `clock`, `bulb`) y llamá a `scrim()` si el fondo detrás de los subtítulos es claro o denso.

## 4. Paridad de Motor y Contrato de Render (`plantilla.html` ↔ `compilador.mjs`)
- Conservá siempre los IDs DOM requeridos (`#frame`, `#cv`, `#subs`, `#taps`, `#pz`, `#replay`, `#bars`, `#info`, `#aside`, `#close`, `#restart`, `#rec`, `#bareSw`, `#script`, `#total`, `#voz`) y el contrato en `window` (`__duration`, `__aspect`, `__escenas`, `renderAt(t)`).
- Mantené paridad de curvas (`ease`, `easeIO`, `q4`, `outExpo`, `inExpo`, `outBack`, `spring`, `pop`), utilidades (`rng`, `gauss`, `mix`, `rgba`), capas (`bleed`, `fixed`, `scrim`, `dot`) e íconos estándar entre `assets/plantilla.html` y `scripts/compilador.mjs`.

## 5. Entorno Windows PowerShell y Compuerta de Verificación
- En Windows con `ExecutionPolicy` restrictiva para scripts `.ps1`, ejecutá `npm` y `npx` siempre mediante `cmd.exe /c "..."`.
- Ningún cambio en escenas, estilos o compilador está completo sin pasar:
  1. `node scripts/simular.cjs <archivo.html> --todos` con **0 errores y 0 avisos** (`9:16`, `16:9`, `1:1`, `4:5`).
  2. Inspección visual de `out/hoja.png` generada con `node scripts/render.mjs <archivo.html> --fotos gancho` y `--fotos finales`.
