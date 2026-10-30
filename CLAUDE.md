# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + HTML5 Canvas. Sin dependencias, sin `package.json`, sin build, sin linter ni tests. El README y el texto de la UI están en español; mantené ese idioma en textos visibles al usuario.

## Ejecutar

Abrir `index.html` directamente en el navegador, o servir la carpeta con cualquier servidor estático (`python -m http.server 8000`, `npx serve .`). La verificación es manual: jugar en el navegador.

## Arquitectura

Tres archivos: `index.html` (canvas `#board` 300×600, canvas `#next-canvas` 120×120, HUD y overlay), `style.css` y `game.js`. Todo el juego vive en `game.js` como un único script con estado global a nivel módulo (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, etc.), cargado con `<script>` clásico y `'use strict'`.

Flujo:
- `init()` resetea todo el estado y arranca el loop; el botón de reinicio y el arranque inicial lo llaman.
- `loop(ts)` (requestAnimationFrame) acumula `dropAccum` y baja la pieza cada `dropInterval` ms; cuando no puede bajar llama a `lockPiece()` → `merge()` → `clearLines()` → `spawn()`.
- `spawn()` promueve `next` a `current`, genera otra con `randomPiece()` y llama a `endGame()` si la nueva pieza colisiona.
- Los inputs (`keydown`) modifican `current` directamente; la colisión se resuelve siempre con `collide(shape, ox, oy)` contra `board`.

Detalles no obvios:
- El tablero es una matriz `ROWS × COLS` de índices de color (0 = vacío, 1–7 = tipo de pieza). Ese mismo índice sirve para `PIECES`, `COLORS` y `drawBlock`, así que agregar una pieza exige tocar los tres en sincronía (y el `* 7` de `randomPiece`).
- `clearLines()` es quien sube `level` y recalcula `dropInterval` (`max(100, 1000 - (level-1)*90)`); el puntaje por líneas se multiplica por `level` ya antes de subirlo.
- Rotación: `rotateCW` + wall kicks simples (`[0, -1, 1, -2, 2]` en x), no es SRS.
- Puntaje de drops: +1 por celda en soft drop, +2 por celda en hard drop.
- Pausa y game over comparten el mismo `#overlay`; ambos reutilizan `overlayTitle`/`overlayScore` y la clase `hidden`.
