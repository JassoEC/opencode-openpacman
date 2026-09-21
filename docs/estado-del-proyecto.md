# Estado del proyecto

> **Propósito:** inventario del estado actual del clon de Pac-Man (qué se puede jugar, cómo está montado y qué queda por hacer), con la feature pendiente más relevante identificada y esbozada a alto nivel.
> **Fecha:** 2026-09-20

## Lo que se puede jugar hoy

Abrir `src/index.html` en el navegador:

- Pac-Man se mueve con las flechas, come dots (+10) y power pellets (+50), pierde vidas al chocar con un fantasma (3 vidas) y gana al vaciar el tablero.
- 4 fantasmas con personalidades clásicas: rojo **hunter** (caza directa), rosa **ambusher** (4 celdas por delante de Pac-Man), cian **flanker** (pivote sobre el hunter) y naranja **shy** (persigue lejos, se retira cerca).
- Los fantasmas salen del corral de forma temporizada y escalonada: naranja (~0 s), cian (~1.5 s), rosa (~3 s), rojo (~4.5 s). Mientras esperan, rebotan verticalmente en el corral.
- 4 power pellets clásicos en las esquinas (1,3), (26,3), (1,23) y (26,23): punto grande parpadeante, 50 puntos, cuentan para la victoria.
- Túnel en la fila 14 con wrap lateral (Pac-Man y fantasmas).

## Arquitectura

HTML único (`src/index.html`) con scripts como globals, sin módulos ni build. **El orden de carga es vinculante**:

1. `src/js/maze.js` → `window.MAZE`, `window.TUNNEL_ROW`, `window.PACMAN_START`, `window.GHOST_STARTS`
2. `src/js/game.js` → `window.createGame`, `window.update`, `window.DIRS`
3. `src/js/render.js` → `window.draw`
4. `src/js/main.js` → bucle (`requestAnimationFrame`), teclado y overlays

Canvas de 560×620 px = 28 cols × 31 filas a `TILE = 20` (`render.js`). El laberinto se codifica como strings de 28 chars (`maze.js`): `#`=pared(1), `.`=dot(2), `-`=puerta del corral(3), espacio=abierto(0), `o`=power pellet(4). `createGame()` copia `MAZE` a `game.grid` para que la partida no mute el original; la puerta (3) bloquea a Pac-Man pero no a los fantasmas.

Sin tests ni linter: la verificación es manual, abriendo el juego y probando casos en la consola del navegador.

## Linea de trabajo (spec-driven)

| Spec | Estado | Contenido |
| --- | --- | --- |
| 01 — Cuatro fantasmas con personalidades | Implementado | 4 kinds, objetivo por personalidad, colores clásicos, elección codiciosa Manhattan. |
| 02 — Salida temporizada del corral | Implementado | `RELEASE_TICKS` (0/1.5/3/4.5 s), rebote vertical en espera, salida dirigida a `PEN_EXIT`. |
| 03 — Power pellets en las esquinas | Implementado | Valor 4 en el grid, +50 pts, parpadeo, cuentan para `dotsRemaining`. |
| 04 y siguientes | — | Pendientes (ver más abajo). |

## Feature pendiente más relevante: modo vulnerable (comer fantasmas)

Tras coger un power pellet, los fantasmas deben volverse comestibles durante unos segundos. Es la pieza central que conecta SPEC 03 (ya hay pellets) con el juego clásico; sin ella el power pellet solo vale 50 puntos.

### Esbozo a alto nivel (no es el spec)

- **Dentro (para el futuro spec 04):**
  - Nuevo estado por fantasma: `vulnerable` durante un tiempo limitado (ej. `FRIGHTENED_TICKS`) que arranca al comer un pellet.
  - Rendering: fantasmas azules mientras son vulnerables, parpadeo en los últimos segundos para avisar del fin del modo.
  - Colisión distinta: Pac-Man *come* a un fantasma vulnerable (vuelve al corral como ojos, sin puntuación o con la puntuación correspondiente) en vez de perder vida.
  - Inversión de dirección de los fantasmas al activarse el modo (comportamiento del arcade).
- **Indefinido todavía:** cadena 200/400/800/1600 por comer fantasmas, respawn con ojos que vuelven al corral, interacción con la salida temporizada de SPEC 02 (que un fantasma comido reentre y vuelva a salir).
- **Fuera:** alternancia scatter/persecución, niveles, dificultad, sonido.

Todo lo durmiente de los specs anteriores es candidato: si se decide entrar, va en su propio spec.

## Roadmap de features aplazadas

Recogidas de la sección "Fuera de alcance" de los specs 01–03 (por si llegan):

- **Modo vulnerable** (el spec 04): fantasmas azules, comestibles, vuelta al corral. ← prioridad.
- Cadena de puntos por comer fantasmas (200/400/800/1600) y parpadeo de aviso del modo.
- Modo scatter (persecución/dispersión) con esquinas de dispersión.
- Niveles: aumento de dificultad, velocidad por fantasma, respawn de pellets.
- Liberación del corral condicionada (dots comidos o posición de Pac-Man) y hunter empezando fuera del corral.
- Sonido.