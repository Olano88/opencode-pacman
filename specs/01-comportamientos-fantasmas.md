# SPEC 01 — Comportamientos de fantasmas

> **Status:** Aprobado
> **Depends on:** ninguno
> **Date:** 2026-09-03
> **Objective:** Implementar 4 fantasmas con los comportamientos clásicos de Pac-Man (Blinky, Pinky, Inky y Clyde), cada uno con su color, para que se comporten de forma distinta y Blinky persiga agresivamente a Pac-Man.

## Scope

**In:**

- Ampliar `GHOST_STARTS` en `src/js/maze.js` a 4 fantasmas (blinky, pinky, inky, clyde), todos dentro de la pen.
- Cuatro conductas fijas (sin temporizador), cada una un cálculo de celda objetivo:
  - `blinky`: la celda de Pac-Man (persecución agresiva).
  - `pinky`: la celda 4 por delante de Pac-Man en su dirección actual.
  - `inky`: objetivo geométrico derivado de la posición de Blinky y de Pac-Man.
  - `clyde`: persigue si está a más de 8 celdas de Pac-Man; si se acerca, se retira a la esquina inferior-izquierda.
- Generalizar `decideGhost` en `src/js/game.js` a un único mecanismo: elegir la dirección que minimiza la distancia Manhattan al objetivo; se mantiene el giro de 180 como salida de callejón.
- Mapear colores clásicos por `kind` en `src/js/render.js`.

**Out of scope (para futuros specs):**

- Alternancia de modos scatter/chase con temporizador.
- Modo frightened / power pellets.
- Salida escalonada de la pen con retardo.
- Cambios en colisión, vidas o puntuación.
- Nombres o indicadores en el HUD.

## Data model

Se reutiliza el modelo existente (celdas enteras al estar `aligned()`, coordenadas origen arriba-izquierda). Cambian dos estructuras y se añade una tabla de objetivos.

```js
// src/js/maze.js
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'blinky' },
  { x: 14, y: 14, kind: 'pinky' },
  { x: 13, y: 15, kind: 'inky' },
  { x: 14, y: 15, kind: 'clyde' },
];
```

```js
// src/js/render.js — sustituye el array indexado por un mapa por kind
const GHOST_COLORS = {
  blinky: '#ff0000',
  pinky:  '#ffb8ff',
  inky:   '#00ffff',
  clyde:  '#ffb852',
};
```

```js
// src/js/game.js — nuevas constantes
const CLYDE_CORNER = { x: 1, y: 29 }; // esquina inferior-izquierda (celda con dot)
const GHOST_TARGETS = { /* una funcion (game, g) => {x, y} por kind */ };
```

Reglas de objetivo:

- `blinky`: `{ x: round(pacman.x), y: round(pacman.y) }`.
- `pinky`: `pacman + 4 * DIRS[pacman.dir]`, recortado a los límites del grid.
- `inky`: `point2 = pacman + 2 * DIRS[pacman.dir]`; objetivo = `point2 + (point2 - blinky)`, recortado a los límites.
- `clyde`: si `manhattan(clyde, pacman) > 8` → celda de Pac-Man; si no → `CLYDE_CORNER`.

## Implementation plan

1. `src/js/maze.js`: sustituir `GHOST_STARTS` por las 4 entradas de arriba. Manual: al abrir la página se ven 4 fantasmas en la pen (aún con colores mixtos).
2. `src/js/render.js`: cambiar `GHOST_COLORS` a mapa por `kind` y dibujar con `GHOST_COLORS[g.kind]`. Manual: cada fantasma muestra su color clásico.
3. `src/js/game.js`: añadir `CLYDE_CORNER` y `GHOST_TARGETS`; reescribir `decideGhost` para que compute el objetivo según `g.kind` y elija la opción con menor distancia, conservando el fallback de 180º. Eliminar la rama `hunter`/`random`. Manual: los 4 se dispersan y Blinky da caza.
4. Prueba de juego completa (ver criterios).

## Acceptance criteria

- [ ] Cargar `src/index.html` no muestra errores en consola.
- [ ] Hay exactamente 4 fantasmas, cada uno con su color clásico (rojo, rosa, cian, naranja).
- [ ] Los 4 arrancan dentro de la pen y salen por la puerta sin quedarse atascados.
- [ ] Con Pac-Man parado, Blinky lo alcanza antes que cualquier otro fantasma partiendo de la pen.
- [ ] En un pasillo recto, Pinky se dirige a una celda 4 por delante de Pac-Man.
- [ ] Mover a Blinky cambia la ruta de Inky (Inky depende de la posición de Blinky).
- [ ] A 8 celdas o menos de Pac-Man, Clyde se retira hacia la esquina inferior-izquierda en vez de perseguir.
- [ ] Comer dots, puntuación, vidas y estados ganar/perder funcionan igual que antes.

## Decisions

- **Sí:** arquetipos clásicos (Blinky/Pinky/Inky/Clyde). Están bien documentados, son verificables y cubren el requisito del "que persigue agresivamente" (Blinky).
- **No:** alternancia scatter/chase con temporizador. Las conductas quedan fijas desde el inicio; la alternancia merece su propio spec.
- **No:** salida escalonada de la pen ni modo frightened. No hay power pellets todavía; se aplaza.
- **Sí:** un único mecanismo de decisión (minimizar distancia al objetivo). Reduce `decideGhost` a un cálculo de objetivo por kind y evita lógica condicional creciente.
- **Sí:** Inky depende de Blinky, como en el arcade original.
- **Sí:** colores por `kind`, no por índice. Evita acoplar `render.js` al orden del array.
- **No:** el quirk histórico de Pinky al mirar hacia arriba (desplazamiento extra). Se usa el simple "4 celdas en la dirección actual".
- **Nota:** los objetivos de Pinky/Inky pueden caer en una pared; es inocuo porque el mecanismo solo elige entre direcciones abiertas.

## Risks

| Risk                                              | Mitigation                                                                 |
| ------------------------------------------------- | -------------------------------------------------------------------------- |
| Objetivo de Pinky/Inky fuera del grid             | Recortar el objetivo a los límites del grid antes de calcular distancias.  |
| Superposición visual de los 4 fantasmas en la pen | Cosmético y pasajero: se separan al salir por tener objetivos distintos.   |

## What is **not** in this spec

- Alternancia de modos scatter/chase (temporizador).
- Modo frightened / power pellets.
- Salida escalonada de la pen con retardo.
- Nombres o indicadores en el HUD.
- Cambios en colisión, vidas o puntuación.

Cada uno de esos, si llega, va en su propio spec.
