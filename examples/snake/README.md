# TITAN Snake 🐍

Snake con estética neón, hecho para la pantalla del teléfono. Toda la lógica
y el dibujo están en `snake.titan`, que se compila a WebAssembly con el
backend `titan wasm`.

- Pantalla vertical 9:16 que se adapta a cualquier teléfono (incluye notch y zonas seguras).
- Controles: **desliza el dedo** (puedes encadenar giros sin levantarlo), **flechas** o **WASD**; para empezar, toca la pantalla o pulsa espacio.
- Efectos: brillo neón, comida que late, explosión de partículas con «+1», sacudida y destello rojo al chocar, línea de escaneo y barra de velocidad.
- La velocidad sube con cada punto. El récord se guarda mientras dure la sesión.
- Al reiniciar se recupera toda la memoria de la partida anterior (`std::wasm::heap_checkpoint` / `heap_restore`).

## Compilar y jugar

```bash
zett wasm examples/snake/snake.titan --output examples/snake/program.wasm
cd examples/snake
python3 -m http.server 8080 --bind 0.0.0.0
```

Abre `http://localhost:8080` en el navegador. Si el servidor corre en Termux,
el mismo teléfono puede abrir esa dirección. Desde otro dispositivo de la
misma red, usa `http://IP-del-equipo:8080`.

## Arquitectura

El backend WebAssembly todavía no tiene variables globales. Por eso el juego
sigue el patrón de Elm: el estado es un valor inmutable `[cabecera, cuerpo]`.

| Export TITAN              | Qué hace                                                        |
|---------------------------|-----------------------------------------------------------------|
| `main()`                  | Ajusta el tamaño del canvas y registra los eventos con `std::web::listen` |
| `init(now)`               | Crea el estado inicial                                          |
| `frame(state, now)`       | Avanza la lógica según el reloj y dibuja el cuadro              |
| `update(state, msg, now)` | Aplica un mensaje de entrada (dirección, toque o deslizamiento) |
| `on_key`, `on_down`, …    | Leen el evento del DOM y devuelven un mensaje entero            |

`host.js` es el adaptador estándar de `examples/browser/host.js` con dos
añadidos: el valor que devuelve un manejador de eventos, si no es cero, se
pasa a `update`; y un bucle `requestAnimationFrame` que llama a `frame`. El
host solo guarda el identificador del estado: no contiene lógica del juego.
