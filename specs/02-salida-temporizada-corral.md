# SPEC 02 — Salida temporizada y escalonada del corral

> **Estado:** Implementado
> **Depende de:** SPEC 01
> **Fecha:** 2026-09-20
> **Objetivo:** Liberar a los fantasmas del corral por temporizador escalonado (0–4.5 s, del menos al más agresivo) de forma independiente al movimiento de Pac-Man, con rebote vertical en la espera y salida a pie por la puerta.

## Motivo

Con Pac-Man quieto, la elección codiciosa Manhattan de `decideGhost` se cicla dentro del corral: subir por la puerta nunca llega a ser la mejor opción intermedia, así que los fantasmas solo escapan cuando Pac-Man se mueve y cambia los objetivos. La liberación por tiempo exige además una salida dirigida a la puerta; el codicioso actual no la garantiza por sí solo.

## Alcance

**Dentro:**

- Constantes nuevas en `src/js/game.js`: `RELEASE_TICKS` (retardo por `kind`), `PEN_EXIT` (celda sobre la puerta) y `PEN` (rectángulo del corral).
- Estado nuevo: `game.tick` (contador de frames en `update`) y `releasedAt` por fantasma.
- Tres casos en `moveGhost` (src/js/game.js): espera (rebote vertical en su celda), liberado dentro del corral (objetivo = `PEN_EXIT` en vez de `ghostTarget`), y fuera del corral (comportamiento de SPEC 01 sin cambios).
- Reinicio de `game.tick` en `resetPositions()` para que cada vida repita la tanda de salidas.

**Fuera de alcance (para specs futuros):**

- Cambios en `maze.js`, `render.js` o `main.js`: el rebote y la salida se resuelven con posición y `dir`, que `drawGhost` ya pinta.
- Liberación condicionada a dots comidos o posición de Pac-Man (counters del arcade).
- El hunter empezando fuera del corral (parte dentro y sale último).
- Power pellets, modo scatter, niveles y dificultad (ya aplazados en SPEC 01).

## Modelo de datos

```js
// src/js/game.js — constantes nuevas (ticks a ~60 fps, como las velocidades)
const RELEASE_TICKS = { shy: 0, flanker: 90, ambusher: 180, hunter: 270 }; // 0 / 1.5 / 3 / 4.5 s
const PEN_EXIT = { x: 13, y: 11 };               // celda sobre la puerta (cols 13-14, fila 12)
const PEN = { x0: 11, x1: 16, y0: 12, y1: 15 };  // corral: interior (filas 13-15) + fila de puerta

// en createGame(): estado nuevo
//   game.tick = 0
//   cada fantasma: releasedAt = RELEASE_TICKS[ g.kind ]
```

Reglas de movimiento por fantasma en `moveGhost`:

- `game.tick < g.releasedAt` → rebote vertical: oscila ±0.5 celdas alrededor de su `y` inicial (14), invirtiendo `dir` en los extremos; no se llama a `decideGhost`.
- Liberado y con celda redondeada dentro de `PEN` → `decideGhost` con objetivo fijo `PEN_EXIT` (13,11).
- Fuera de `PEN` → objetivo de `ghostTarget` según su `kind` (SPEC 01, intacto).

`resetPositions()` (game.js) reinicia `game.tick = 0`; los `releasedAt` derivan del `kind` y no cambian.

## Plan de implementación

1. `game.js`: añadir `RELEASE_TICKS`, `PEN_EXIT` y `PEN`; en `createGame()` añadir `tick: 0` al juego y `releasedAt` a cada fantasma. Prueba manual: el juego arranca y se juega igual que antes, sin errores en consola.
2. `game.js`: en `update()` incrementar `game.tick`; en `moveGhost()` los tres casos (rebote / salida a `PEN_EXIT` / AI de SPEC 01). Prueba manual: sin tocar teclas, salen naranja (~0 s), cian (~1.5 s), rosa (~3 s) y rojo (~4.5 s), todos por la puerta.
3. `game.js`: en `resetPositions()` reiniciar `game.tick = 0`. Prueba manual: perder una vida y comprobar que la tanda se repite (naranja primero, rojo a los ~4.5 s).

## Criterios de aceptación

- [ ] Al abrir `src/index.html` no hay errores en la consola del navegador.
- [ ] Sin tocar ninguna tecla, los 4 fantasmas abandonan el corral por la puerta en 5 s o menos.
- [ ] La liberación no depende del movimiento de Pac-Man: con Pac-Man quieto toda la partida, salen igual.
- [ ] El orden de salida es naranja (shy) → cian (flanker) → rosa (ambusher) → rojo (hunter), con el rojo a los ~4.5 s.
- [ ] Ningún fantasma queda ciclado dentro del corral tras su liberación.
- [ ] Mientras esperan, los fantasmas oscilan verticalmente en su celda sin salir del corral (cols 11-16, filas 12-15).
- [ ] Al perder una vida, los 4 vuelven al corral y la tanda de salidas se reinicia completa.
- [ ] Una vez fuera, cada fantasma conserva su personalidad de SPEC 01 (el rojo persigue directo; el naranja se retira a ≤ 8 celdas de Pac-Man).

## Decisiones

- **Sí:** liberación solo por tiempo (`game.tick`), independiente de Pac-Man. Es el requisito explícito del usuario y elimina el acoplamiento actual.
- **Sí:** orden naranja → cian → rosa → rojo con espaciado 0/1.5/3/4.5 s. Decisión del usuario: presión creciente del menos al más agresivo; el hunter cierra a los 4.5 s, dentro de la ventana 3-5 s pedida.
- **Sí:** rebote vertical en la celda mientras esperan. Aspecto arcade clásico a coste mínimo; los ojos de `drawGhost` ya siguen `dir` (render sin cambios).
- **Sí:** salida caminando hacia `PEN_EXIT` hasta estar fuera. El codicioso actual se cicla dentro del corral; una salida dirigida la garantiza sin saltos visibles.
- **Sí:** contador de frames en el estado del juego. Todo el juego ya avanza por frame a ~60 fps (velocidades incluidas); `performance.now()` desincronizaría con el movimiento existente.
- **Sí:** reiniciar `game.tick` en `resetPositions()`. Cada vida concede el mismo margen; evita la muerte instantánea al reaparecer.
- **No:** teletransporte a la puerta al liberarse. Salto visible en pantalla; descartado en la fase de preguntas.
- **No:** liberación por dots comidos o posición de Pac-Man (mecánica del arcade original). Complejidad extra sin valor para este objetivo.
- **No:** tocar `maze.js`, `render.js` o `main.js`. No hacen falta: el estado nuevo vive en `game` y el rebote se dibuja solo.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| `requestAnimationFrame` a >60 fps acorta los segundos reales | Constante del proyecto: las velocidades de SPEC 01 ya son por frame; los retardos viven en una sola constante (`RELEASE_TICKS`) ajustable. |
| Liberación a mitad del rebote (posición fraccionaria) | La alineación existente de `moveGhost` lo absorbe: el fantasma termina el paso en curso (≤ 5 frames) antes de decidir hacia la puerta. |

## Qué **no** entra en este spec

- Liberación por dots comidos o posición de Pac-Man.
- El hunter empezando fuera del corral.
- Power pellets, modo scatter, niveles, dificultad creciente.

Cada uno, si llega, va en su propio spec.
