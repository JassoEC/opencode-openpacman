# SPEC 01 — Cuatro fantasmas con personalidades clásicas

> **Estado:** Implementado
> **Depende de:** Ninguna (primer spec del proyecto)
> **Fecha:** 2026-09-20
> **Objetivo:** Añadir 4 fantasmas al juego, cada uno con su propia personalidad arcade clásica (cazador agresivo, emboscador, flanqueador y tímido), distinguibles por color.

## Alcance

**Dentro:**

- Ampliar `GHOST_STARTS` (src/js/maze.js) de 2 a 4 fantasmas, todos dentro del corral, cada uno con su `kind`: `hunter`, `ambusher`, `flanker`, `shy`.
- Reescribir la decisión de dirección en `decideGhost` (src/js/game.js): cada fantasma calcula una celda objetivo según su `kind` y elige codiciosamente la dirección que más se acerca a ella.
- Asignar los colores clásicos por personalidad en `GHOST_COLORS` (src/js/render.js): rojo = cazador, rosa = emboscador, cian = flanqueador, naranja = tímido.
- Eliminar el `kind: 'random'`: los 4 fantasmas tienen comportamiento definido.

**Fuera de alcance (para specs futuros):**

- Power pellets y modo vulnerable (fantasmas comestibles).
- Alternancia persecución/dispersión (modo scatter) y esquinas de dispersión (salvo la esquina de retirada del tímido).
- Salida escalonada del corral con temporizadores.
- Niveles, aumento de dificultad, velocidad por fantasma, puntuación por comer fantasmas.

## Modelo de datos

```js
// src/js/maze.js — los 4 dentro del corral (interior: cols 11-16, filas 13-15)
const GHOST_STARTS = [
  { x: 12, y: 14, kind: 'hunter' },   // rojo: caza directa (el agresivo)
  { x: 13, y: 14, kind: 'ambusher' }, // rosa: 4 celdas delante de Pac-Man
  { x: 14, y: 14, kind: 'flanker' },  // cian: pivote sobre el hunter
  { x: 15, y: 14, kind: 'shy' },      // naranja: persigue lejos, se retira cerca
];
```

Reglas de celda objetivo por `kind` (src/js/game.js), con `p` = pacman y `P` = punto a 2 celdas delante de Pac-Man:

- `hunter`: objetivo = celda actual de Pac-Man.
- `ambusher`: objetivo = celda de Pac-Man + 4 × su vector de dirección.
- `flanker`: objetivo = `P + (P − hunter)` (vector desde el hunter duplicado, como Inky).
- `shy`: si distancia euclídea a Pac-Man > `SHY_THRESHOLD` (8 celdas) → objetivo = celda de Pac-Man; si ≤ 8 → objetivo = `SHY_CORNER` (celda (1, 29), esquina inferior izquierda).

Constantes nuevas en game.js: `SHY_THRESHOLD = 8`, `SHY_CORNER = { x: 1, y: 29 }`.

Elección de dirección (igual que el hunter actual, generalizada a todos): entre las direcciones válidas (sin 180° salvo callejón sin salida), la que minimiza la distancia Manhattan desde la celda siguiente al objetivo.

```js
// src/js/render.js — orden ligado por índice a GHOST_STARTS
const GHOST_COLORS = [ '#ff0000', '#ffb8ff', '#00ffff', '#ffb852' ];
```

## Plan de implementación

1. `maze.js`: ampliar `GHOST_STARTS` a las 4 entradas anteriores. Prueba manual: se ven 4 fantasmas en el corral y el juego arranca sin errores (los kinds nuevos caen temporalmente en la rama random).
2. `game.js`: añadir `ghostTarget( game, g )` con las 4 reglas de objetivo y las constantes `SHY_THRESHOLD` / `SHY_CORNER`.
3. `game.js`: `decideGhost` usa `ghostTarget` y la elección codiciosa Manhattan; se elimina la rama `random`. Prueba manual: el rojo persigue a Pac-Man, el rosa le corta por delante.
4. `render.js`: reordenar `GHOST_COLORS` al mapeo clásico. Prueba manual: rojo = agresivo, rosa = emboscador, cian = flanqueador, naranja = tímido.

## Criterios de aceptación

- [x] Al abrir `src/index.html` no hay errores en la consola del navegador.
- [x] Se ven exactamente 4 fantasmas: uno rojo, uno rosa, uno cian y uno naranja.
- [x] Los 4 fantasmas salen del corral por la puerta sin quedarse atrapados.
- [x] El fantasma rojo se dirige hacia la celda actual de Pac-Man en cada intersección.
- [x] El fantasma rosa se dirige hacia un punto por delante de Pac-Man, no hacia su celda actual.
- [x] Con la misma posición de Pac-Man, `ghostTarget` devuelve objetivos distintos para cada `kind` (verificable en consola).
- [x] El fantasma naranja persigue estando a más de 8 celdas y se retira hacia la esquina inferior izquierda estando a 8 o menos.
- [x] El giro de 180° solo ocurre en callejones sin salida, nunca como opción normal.
- [x] Al chocar con cualquier fantasma se pierde una vida y los 4 vuelven a sus posiciones iniciales del corral.
- [x] Al perder las 3 vidas aparece PERDISTE; al comer todos los dots, GANASTE.

## Decisiones

- **Sí:** personalidades arcade clásicas (Blinky/Pinky/Inky/Clyde). Son 4 comportamientos distintos, documentados y equilibrados: exactamente lo que pide el objetivo.
- **Sí:** kinds semánticos en inglés (`hunter`, `ambusher`, `flanker`, `shy`). Coherentes con los identificadores existentes (`hunter`, `createGame`); los comentarios siguen en español.
- **Sí:** agresividad = caza directa codiciosa con la misma velocidad que los demás. La agresividad es la persecución constante, no la velocidad.
- **No:** velocidad por fantasma. Eje de dificultad no pedido; pertenece a un futuro spec de niveles.
- **Sí:** pivote del flanqueador sobre la posición real del `hunter`. Fiel al arcade; `game.ghosts` ya es accesible en `decideGhost`.
- **No:** replicar el bug de overflow del arcade al mirar hacia arriba. Complejidad sin valor jugable aquí.
- **Sí:** los 4 dentro del corral con salida libre. Sin temporizadores ni estado nuevo.
- **Sí:** eliminar `kind: 'random'`. Los 4 tienen comportamiento definido; no quedan ramas muertas.
- **Sí:** Manhattan para elegir dirección (mecánica existente) y euclídea solo para el umbral del tímido (fiel al original).
- **Sí:** colores ligados por índice a `GHOST_STARTS` reordenando `GHOST_COLORS`. Cambio mínimo en render.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| El tímido con umbral 8 puede resultar inofensivo en un tablero 28×31 | Umbral y esquina en constantes (`SHY_THRESHOLD`, `SHY_CORNER`), ajustables sin tocar la lógica. |
| Los 4 salen a la vez y se solapan | Aceptable: el arcade también los solapa; no hay colisión fantasma-fantasma. |
| 4 perseguidores pueden hacer el juego demasiado difícil | Las personalidades dispersan los objetivos; si se confirma, se ajusta en el futuro spec de dificultad. |

## Qué **no** entra en este spec

- Power pellets y modo vulnerable.
- Modo scatter (persecución/dispersión).
- Salida escalonada del corral.
- Niveles, dificultad creciente, velocidad por fantasma, puntos por comer fantasmas.

Cada uno, si llega, va en su propio spec.
