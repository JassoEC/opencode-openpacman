# SPEC 03 — Cuatro power pellets clásicos en las esquinas

> **Estado:** Approved
> **Depende de:** Ninguna (solo el juego base; no toca fantasmas)
> **Fecha:** 2026-09-20
> **Objetivo:** Añadir 4 power pellets clásicos (punto grande parpadeante de 50 puntos) en las cuatro esquinas del tablero, contando para la victoria igual que los dots.

## Alcance

**Dentro:**

- Nuevo carácter `o` en `MAZE_STR` (src/js/maze.js), parseado a valor 4, en las 4 esquinas clásicas: (1,3), (26,3), (1,23), (26,23).
- Comer pellet en `movePacman` (src/js/game.js): +50 puntos, celda a 0, decrementa `dotsRemaining`.
- `dotsRemaining` cuenta valores 2 y 4 en `createGame()`: GANASTE exige comer también los 4 pellets.
- `drawDots` (src/js/render.js) dibuja el valor 4 como punto grande (~7 px de radio) que parpadea según `frame`.

**Fuera de alcance (para specs futuros):**

- Modo vulnerable: fantasmas azules, comestibles, vuelta al corral al ser comidos.
- Cadena 200/400/800/1600 por comer fantasmas, parpadeo de aviso del modo, inversión de dirección al activarse.
- Niveles, respawn de pellets, sonido o animación al comerlos.

## Modelo de datos

```js
// src/js/maze.js — filas 3 y 23 con 'o' en las esquinas
' #o####.#####.##.#####.####o# ' // 3
' #o..##................##..o# ' // 23  fila inicio Pacman

// parseTile: if ( ch === 'o' ) return 4; // power pellet
```

- Valor 4 = power pellet: transitable (no está en `isWall`), se come como un dot.
- Sin estado nuevo: el pellet vive en `game.grid` y muere al comerlo (0), igual que los dots.
- `createGame()`: `dots++` pasa a contar `v === 2 || v === 4`.
- `drawDots( ctx, grid, frame )`: firma nueva con `frame`; el pellet es visible cuando `( frame % 20 ) < 10` (~0.33 s por fase).

## Plan de implementación

1. `maze.js`: añadir `o` a `parseTile` y sustituir las 4 celdas de `MAZE_STR`. Prueba manual: el juego arranca y se juega igual, sin errores en consola (el valor 4 es transitable aunque aún no se pinte ni se coma).
2. `game.js`: contar 2 y 4 en `dotsRemaining` y comer el 4 en `movePacman` (+50, celda a 0, decremento). Prueba manual: en consola `game.grid[3][1] === 4`; al pasar por una esquina el SCORE sube 50 y la celda pasa a 0.
3. `render.js`: `drawDots` recibe `frame` y pinta el valor 4 como círculo grande parpadeante. Prueba manual: 4 puntos grandes parpadeando cerca de las esquinas que desaparecen al comerlos.

## Criterios de aceptación

- [ ] Al abrir `src/index.html` no hay errores en la consola del navegador.
- [ ] Se ven exactamente 4 power pellets en (1,3), (26,3), (1,23), (26,23).
- [ ] Cada pellet es un punto claramente mayor que un dot normal y parpadea (alterna visible/invisible).
- [ ] Comer un power pellet suma exactamente 50 puntos (verificable en el HUD).
- [ ] El pellet comido no reaparece al perder una vida (mismo comportamiento que los dots).
- [ ] El `dotsRemaining` inicial cuenta dots + pellets (verificable en consola).
- [ ] GANASTE solo aparece tras comer todos los dots y los 4 pellets.
- [ ] Comer un pellet no altera el comportamiento de los fantasmas.
- [ ] Pac-Man atraviesa la celda del pellet con normalidad (no es muro).

## Decisiones

- **Sí:** solo los pellets, sin modo vulnerable. Decisión del usuario en fase de preguntas; el modo vulnerable va en su propio spec.
- **Sí:** carácter `o` → valor 4 en el grid. Consistente con la codificación por tiles; la copia por partida de `createGame()` aísla el estado sin código nuevo.
- **Sí:** 50 puntos y cuentan para la victoria. Fiel al arcade.
- **Sí:** posiciones (1,3), (26,3), (1,23), (26,23). Las del nivel 1 original, simétricas; la fila 23 es la de inicio de Pac-Man.
- **Sí:** parpadeo con `frame % 20 < 10`. Apariencia arcade sin estado nuevo: `draw` ya recibe `frame`.
- **No:** array `POWER_PELLETS` aparte. Duplicaría el estado (posición + comido) sin beneficio.
- **No:** modo vulnerable, cadena de puntos por fantasmas, inversión de dirección. Fuera del objetivo; spec propio.
- **No:** sonido. No existe audio en el proyecto; sería otro eje.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Pellet invisible durante la fase "off" del parpadeo | Ciclo corto (~0.33 s por fase); el arcade original parpadea igual. |
| `drawDots` cambia de firma (parámetro `frame`) | Único punto de llamada es `draw()` en el mismo archivo. |

## Qué **no** entra en este spec

- Modo vulnerable (fantasmas comestibles) y todo su comportamiento.
- Puntos por comer fantasmas (200/400/800/1600).
- Niveles, respawn de pellets, sonido.

Cada uno, si llega, va en su propio spec.
