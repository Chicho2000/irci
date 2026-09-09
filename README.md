# Trabajos sobre RTM32 / STX4

Este repositorio contiene documentación y programas para la CPU virtual
RTM32/STX4:

- [Tests de instrucciones](test-inst-rtm32.md): conjunto de 19 casos de prueba para verificar distintas instrucciones de la CPU.
- [Snake en assembler](snake/snake.rtm): implementación del juego Snake para ejecutarse en la CPU RTM32/STX4.

## Tests de la CPU

Las pruebas de `test-inst-rtm32.md` comprueban instrucciones aritméticas, lógicas, de acceso a memoria, saltos, branches y llamadas a subrutinas. Cada caso documenta las condiciones iniciales, la codificación del programa, el resultado esperado y las conclusiones obtenidas.

Las instrucciones se cargaron manualmente en la memoria de programa mediante su codificación de máquina y se ejecutaron usando el debugger interactivo. Las instrucciones `TRAP` y `RFT` no se incluyen porque dependen del mecanismo de excepciones de la CPU.

## Snake en assembler

El archivo `snake/snake.rtm` contiene un juego interactivo implementado en assembler RTM32/STX4. El programa:

- dibuja el juego en la consola mediante secuencias ANSI;
- lee las teclas `W`, `A`, `S` y `D` desde el dispositivo de entrada;
- mueve una serpiente de hasta tres segmentos;
- detecta los límites del tablero y las colisiones con el cuerpo;
- permite comer dos elementos para ganar.

La entrada y la salida se realizan mediante MMIO, usando la dirección `0xFFFFFF00` (representada en el código por `-0x100`). El programa finaliza mostrando `GANASTE!` o `PERDISTE!` y ejecutando `TRAP 1`.

### Ejecución

Para ejecutar Snake se necesita una CPU RTM32/STX4 o su emulador con soporte para assembler, consola, teclado y MMIO. Cargá `snake/snake.rtm`, iniciá el programa y controlá la serpiente con `W`, `A`, `S` y `D`.
