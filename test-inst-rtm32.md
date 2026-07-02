# Tests de instrucciones RTM32 / STX4

Este archivo contiene pruebas realizadas sobre la CPU virtual RTM32/STX4.

Como actualmente no se cuenta con assembler, las instrucciones fueron cargadas manualmente en memoria usando su codificación correspondiente.

El objetivo de cada caso es verificar si una instrucción funciona correctamente observando los cambios producidos en registros o memoria después de ejecutarla.
---

# Caso 1

## Descripción

Testeo de `ADD`.

Se espera:

```
R[12] = R[10] + R[11]
R[12] = 5 + 7 = 12
```

## Instructions

- `ADD`

## Precondiciones

- `R10 = 5`
- `R11 = 7`
- `R12 = 0`
- Instrucción cargada en `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucción:

```
ADD $10, $11, $12
```

Bits completos:

```
00000010100101101100000000011100
```

Comandos usados:

```
s r10 5
s r11 7
s r12 0
s [0x0] 0x0296C01C
s pc 0x0
r
step
r
```

## Postcondiciones

Resultado observado:

```
R[10]: 0x00000005
R[11]: 0x00000007
R[12]: 0x0000000C
PC   : 0x00000004
```

## Conclusiones

La instrucción `ADD` funcionó correctamente porque `R12` quedó en `0x0000000C`, que equivale a `12` en decimal.

---

# Caso 2

## Descripción

Testeo de `SUB`.

Se espera:

```
R[12] = R[10] - R[11]
R[12] = 20 - 8 = 12
```

## Instructions

- `SUB`

## Precondiciones

- `R10 = 20`
- `R11 = 8`
- `R12 = 0`
- Instrucción cargada en `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucción:

```
SUB $10, $11, $12
```

Bits completos:

```
00000010100101101100000000011101
```

Comandos usados:

```
s r10 20
s r11 8
s r12 0
s [0x0] 0x0296C01D
s pc 0x0
r
step
r
```

## Postcondiciones

Resultado observado:

```
R[10]: 0x00000014
R[11]: 0x00000008
R[12]: 0x0000000C
PC   : 0x00000004
```

Última operación de memoria observada:

```
Last Memory Operation:
Address: 0x00000000 | Size: 0x00000004 | Type: FETCH
```

## Conclusiones

La instrucción `SUB` funcionó correctamente porque `R12` quedó en `0x0000000C`.

---

# Caso 3

## Descripción

Testeo conjunto de instrucciones lógicas tipo R usando los mismos registros fuente.

En este caso se prueban `AND`, `OR`, `XOR` y `NOR`.

Valores usados:

```
R10 = 12
R11 = 10
```

En binario:

```
12 = 1100
10 = 1010
```

Resultados esperados:

```
AND: 1100 & 1010 = 1000 = 8
OR : 1100 | 1010 = 1110 = 14
XOR: 1100 ^ 1010 = 0110 = 6
NOR: NOT(1100 | 1010) = NOT(1110)
```

Como la CPU trabaja con registros de 32 bits, el resultado de `NOR` queda con todos los bits invertidos:

```
NOR = NOT(0x0000000E)
NOR = 0xFFFFFFF1
```

## Instructions

- `AND`
- `OR`
- `XOR`
- `NOR`

## Precondiciones

- `R10 = 12`
- `R11 = 10`
- `R13 = 0`
- `R14 = 0`
- `R15 = 0`
- `R16 = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
AND $10, $11, $13
OR  $10, $11, $14
XOR $10, $11, $15
NOR $10, $11, $16
```

Comandos usados:

```
reset
s r10 12
s r11 10
s r13 0
s r14 0
s r15 0
s r16 0
s [0x0] 0x0296D008
s [0x4] 0x0296E009
s [0x8] 0x0296F00A
s [0xC] 0x0297000B
s pc 0x0
step 4
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[10]: 0x0000000C
R[11]: 0x0000000A
R[13]: 0x00000008
R[14]: 0x0000000E
R[15]: 0x00000006
R[16]: 0xFFFFFFF1
PC   : 0x00000010
```

## Conclusiones

Las instrucciones `AND`, `OR`, `XOR` y `NOR` funcionaron correctamente.

`AND` guardó en `R13` el valor `8`.

`OR` guardó en `R14` el valor `14`.

`XOR` guardó en `R15` el valor `6`.

`NOR` guardó en `R16` el valor `0xFFFFFFF1`, que corresponde a negar bit a bit el resultado de `12 OR 10`.

Los resultados coinciden con lo esperado para los valores `R10 = 12` y `R11 = 10`.

---

# Caso 4

## Descripción

Testeo conjunto de instrucciones de desplazamiento tipo R, donde la cantidad de desplazamiento se toma desde un registro.

En este caso se prueban `SLLR`, `SRLR` y `SRAR`.

Valores usados:

```
R10 = 2
R11 = 16
R12 = 0xFFFFFFF0
```

Se usa `R10` como registro que contiene la cantidad de desplazamiento.

Resultados esperados:

```
SLLR: R13 = R11 << R10[4:0]
SLLR: R13 = 16 << 2 = 64 = 0x00000040

SRLR: R14 = R11 >> R10[4:0]
SRLR: R14 = 16 >> 2 = 4 = 0x00000004

SRAR: R15 = R12 >>> R10[4:0]
SRAR: R15 = -16 >>> 2 = -4 = 0xFFFFFFFC
```

## Instructions

- `SLLR`
- `SRLR`
- `SRAR`

## Precondiciones

- `R10 = 2`
- `R11 = 16`
- `R12 = 0xFFFFFFF0`
- `R13 = 0`
- `R14 = 0`
- `R15 = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
SLLR $10, $11, $13
SRLR $10, $11, $14
SRAR $10, $12, $15
```

Comandos usados:

```
reset
s r10 2
s r11 16
s r12 0xFFFFFFF0
s r13 0
s r14 0
s r15 0
s [0x0] 0x0296D003
s [0x4] 0x0296E004
s [0x8] 0x0298F005
s pc 0x0
r
n 3
r
```

## Postcondiciones

Resultado observado:

```
R[10]: 0x00000002
R[11]: 0x00000010
R[12]: 0xFFFFFFF0
R[13]: 0x00000040
R[14]: 0x00000004
R[15]: 0xFFFFFFFC
PC   : 0x0000000C
```

Última operación de memoria observada:

```
Last Memory Operation:
Address: 0x00000008 | Size: 0x00000004 | Type: FETCH
```

## Conclusiones

Las instrucciones `SLLR`, `SRLR` y `SRAR` funcionaron correctamente.

`SLLR` guardó en `R13` el valor `0x00000040`, que equivale a `64`.

`SRLR` guardó en `R14` el valor `0x00000004`, que equivale a `4`.

`SRAR` guardó en `R15` el valor `0xFFFFFFFC`, que equivale a `-4` en complemento a dos.

El `PC` quedó en `0x0000000C`, lo que indica que se ejecutaron 3 instrucciones de 32 bits desde `0x0`, `0x4` y `0x8`.

---

# Caso 5

## Descripción

Testeo conjunto de instrucciones aritméticas tipo R.

En este caso se prueban `MUL`, `MULH`, `MULHU`, `DIV`, `DIVU`, `REST` y `RESTU`.

Valores usados para multiplicación:

```
R10 = 0xFFFFFFFF
R11 = 2
```

Estos valores permiten diferenciar multiplicación con signo y sin signo:

```
0xFFFFFFFF como signed   = -1
0xFFFFFFFF como unsigned = 4294967295
```

Resultados esperados para multiplicación:

```
MUL:   R13 = parte baja de (R10 * R11)
MUL:   R13 = parte baja de (0xFFFFFFFF * 2) = 0xFFFFFFFE

MULH:  R14 = parte alta signed de (R10 * R11)
MULH:  R14 = parte alta de (-1 * 2) = parte alta de -2 = 0xFFFFFFFF

MULHU: R15 = parte alta unsigned de (R10 * R11)
MULHU: R15 = parte alta de (4294967295 * 2) = 0x00000001
```

Valores usados para división y resto:

```
R20 = 20
R21 = 6
```

Resultados esperados para división y resto:

```
DIV:   R22 = R20 / R21 = 20 / 6 = 3
DIVU:  R23 = R20 / R21 = 20 / 6 = 3

REST:  R24 = R20 % R21 = 20 % 6 = 2
RESTU: R25 = R20 % R21 = 20 % 6 = 2
```

## Instructions

- `MUL`
- `MULH`
- `MULHU`
- `DIV`
- `DIVU`
- `REST`
- `RESTU`

## Precondiciones

- `R10 = 0xFFFFFFFF`
- `R11 = 2`
- `R13 = 0`
- `R14 = 0`
- `R15 = 0`
- `R20 = 20`
- `R21 = 6`
- `R22 = 0`
- `R23 = 0`
- `R24 = 0`
- `R25 = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
MUL   $10, $11, $13
MULH  $10, $11, $14
MULHU $10, $11, $15

DIV   $20, $21, $22
DIVU  $20, $21, $23
REST  $20, $21, $24
RESTU $20, $21, $25
```

Comandos usados:

```
reset
s r10 0xFFFFFFFF
s r11 2
s r13 0
s r14 0
s r15 0
s r20 20
s r21 6
s r22 0
s r23 0
s r24 0
s r25 0
s [0x0] 0x0296D015
s [0x4] 0x0296E016
s [0x8] 0x0296F017
s [0xC] 0x052B6018
s [0x10] 0x052B7019
s [0x14] 0x052B801A
s [0x18] 0x052B901B
s pc 0x0
r
n 7
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[10]: 0xFFFFFFFF
R[11]: 0x00000002
R[13]: 0xFFFFFFFE
R[14]: 0xFFFFFFFF
R[15]: 0x00000001

R[20]: 0x00000014
R[21]: 0x00000006
R[22]: 0x00000003
R[23]: 0x00000003
R[24]: 0x00000002
R[25]: 0x00000002

PC   : 0x0000001C
```

## Conclusiones

Las instrucciones `MUL`, `MULH`, `MULHU`, `DIV`, `DIVU`, `REST` y `RESTU` funcionaron correctamente si los registros destino quedaron con los valores esperados.

`MUL` guardó en `R13` la parte baja del producto, dando `0xFFFFFFFE`.

`MULH` guardó en `R14` la parte alta del producto interpretado con signo, dando `0xFFFFFFFF`.

`MULHU` guardó en `R15` la parte alta del producto interpretado sin signo, dando `0x00000001`.

`DIV` guardó en `R22` el cociente de `20 / 6`, dando `3`.

`DIVU` guardó en `R23` el cociente sin signo de `20 / 6`, dando `3`.

`REST` guardó en `R24` el resto de `20 % 6`, dando `2`.

`RESTU` guardó en `R25` el resto sin signo de `20 % 6`, dando `2`.

El `PC` quedó en `0x0000001C`, lo que indica que se ejecutaron 7 instrucciones de 32 bits desde `0x0` hasta `0x18`.

---

# Caso 6

## Descripción

Testeo conjunto de instrucciones de desplazamiento tipo R con desplazamiento inmediato.

En este caso se prueban `SLL`, `SRL` y `SRA`.

Valores usados:

```
R11 = 16
R12 = 0xFFFFFFF0
aux = 2
```

Resultados esperados:

```
SLL: R13 = R11 << 2
SLL: R13 = 16 << 2 = 64 = 0x00000040

SRL: R14 = R11 >> 2
SRL: R14 = 16 >> 2 = 4 = 0x00000004

SRA: R15 = R12 >>> 2
SRA: R15 = -16 >>> 2 = -4 = 0xFFFFFFFC
```

## Instructions

- `SLL`
- `SRL`
- `SRA`

## Precondiciones

- `R11 = 16`
- `R12 = 0xFFFFFFF0`
- `R13 = 0`
- `R14 = 0`
- `R15 = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
SLL $0,  $11, $13, 2
SRL $0,  $11, $14, 2
SRA $12, $0,  $15, 2
```

Comandos usados:

```
reset
s r11 16
s r12 0xFFFFFFF0
s r13 0
s r14 0
s r15 0
s [0x0] 0x0016D100
s [0x4] 0x0016E101
s [0x8] 0x0300F102
s pc 0x0
r
n 3
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[11]: 0x00000010
R[12]: 0xFFFFFFF0
R[13]: 0x00000040
R[14]: 0x00000004
R[15]: 0xFFFFFFFC
PC   : 0x0000000C
```

## Conclusiones

Las instrucciones `SLL`, `SRL` y `SRA` funcionaron correctamente si los registros destino quedaron con los valores esperados.

`SLL` guardó en `R13` el valor `0x00000040`, que equivale a `64`.

`SRL` guardó en `R14` el valor `0x00000004`, que equivale a `4`.

`SRA` guardó en `R15` el valor `0xFFFFFFFC`, que equivale a `-4` en complemento a dos.

El `PC` quedó en `0x0000000C`, lo que indica que se ejecutaron 3 instrucciones de 32 bits desde `0x0`, `0x4` y `0x8`.

---

# Caso 7

## Descripción

Testeo conjunto de instrucciones tipo R para mover datos entre registros generales y registros especiales.

En este caso se prueban `CFS` y `CTS`.

`CFS` debería copiar un registro especial hacia un registro general.

`CTS` debería copiar un registro general hacia un registro especial.

Para la prueba se intentó usar el registro especial `VBR`.

Valores usados:

```
R10 = 0
R11 = 0xF0000100
VBR = 0xF0000000
```

Resultados esperados:

```
CFS: R10 = VBR
CFS: R10 = 0xF0000000

CTS: VBR = R11
CTS: VBR = 0xF0000100
```

## Instructions

- `CFS`
- `CTS`

## Precondiciones

- `R10 = 0`
- `R11 = 0xF0000100`
- `VBR = 0xF0000000`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones usadas como referencia:

```
CFS $10, 4
CTS $11, 4
```

Comandos usados:

```
reset
s r10 0
s r11 0xF0000100
s [0x0] 0x02800206
s [0x4] 0x02C00207
s pc 0x0
r
n 2
r
```

## Postcondiciones

Resultado observado antes de ejecutar:

```
R[10]: 0x00000000
R[11]: 0xF0000100
VBR  : 0xF0000000
PC   : 0x00000000
```

Resultado observado después de ejecutar:

```
R[10]: 0x00000000
R[11]: 0xF0000100
VBR  : 0xF0000000
PC   : 0x00000008
```

Última operación de memoria observada:

```
Last Memory Operation:
Address: 0x00000004 | Size: 0x00000004 | Type: FETCH
```

## Conclusiones

Las instrucciones `CFS` y `CTS` no dieron el resultado esperado en esta prueba.

Se esperaba que `CFS` copiara el valor de `VBR` hacia `R10`, dejando:

```
R10 = 0xF0000000
```

Pero `R10` quedó en:

```
R10 = 0x00000000
```

También se esperaba que `CTS` copiara el valor de `R11` hacia `VBR`, dejando:

```
VBR = 0xF0000100
```

Pero `VBR` quedó en:

```
VBR = 0xF0000000
```

El `PC` sí avanzó hasta `0x00000008`, por lo que las dos instrucciones fueron buscadas y ejecutadas desde memoria, pero no produjeron los cambios esperados.

Por lo tanto, este caso queda registrado como fallido o inconcluso.

---

# Caso 8

## Descripción

Testeo conjunto de instrucciones de comparación tipo R.

En este caso se prueban `SLT` y `SLTU`.

`SLT` compara con signo, mientras que `SLTU` compara sin signo.

Valores usados:

```
R10 = 0xFFFFFFFF
R11 = 1
```

Estos valores permiten diferenciar comparación con signo y sin signo:

```
0xFFFFFFFF como signed   = -1
0xFFFFFFFF como unsigned = 4294967295
```

Resultados esperados:

```
SLT:  R12 = (R10 < R11) con signo
SLT:  R12 = (-1 < 1) = 1

SLTU: R13 = (R10 < R11) sin signo
SLTU: R13 = (4294967295 < 1) = 0
```

## Instructions

- `SLT`
- `SLTU`

## Precondiciones

- `R10 = 0xFFFFFFFF`
- `R11 = 1`
- `R12 = 0`
- `R13 = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
SLT  $10, $11, $12
SLTU $10, $11, $13
```

Comandos usados:

```
reset
s r10 0xFFFFFFFF
s r11 1
s r12 0
s r13 0
s [0x0] 0x0296C00C
s [0x4] 0x0296D00D
s pc 0x0
r
n 2
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[10]: 0xFFFFFFFF
R[11]: 0x00000001
R[12]: 0x00000001
R[13]: 0x00000000
PC   : 0x00000008
```

## Conclusiones

Las instrucciones `SLT` y `SLTU` funcionaron correctamente si los registros destino quedaron con los valores esperados.

`SLT` guardó en `R12` el valor `1`, porque interpreta `0xFFFFFFFF` como `-1`, y `-1 < 1` es verdadero.

`SLTU` guardó en `R13` el valor `0`, porque interpreta `0xFFFFFFFF` como `4294967295`, y `4294967295 < 1` es falso.

El `PC` quedó en `0x00000008`, lo que indica que se ejecutaron 2 instrucciones de 32 bits desde `0x0` y `0x4`.

---

# Caso 9

## Descripción

Testeo conjunto de instrucciones de salto tipo R.

En este caso se prueban `JR` y `JALR`.

`JR` salta a la dirección guardada en un registro.

`JALR` salta a la dirección guardada en un registro y además guarda la dirección de retorno. Inicialmente probé una codificación incorrecta para `JALR`, lo cual permitió detectar que en esta CPU el registro donde se guarda el retorno se toma del campo `rd`.

Valores usados:

```
R10 = 0x8
R11 = 0x10
R31 = 0
```

Resultados esperados finales:

```
JR:   PC = R10
JR:   PC = 0x8

JALR: R31 = PC + 4
JALR: R31 = 0x8 + 4 = 0xC

JALR: PC = R11
JALR: PC = 0x10
```

## Instructions

- `JR`
- `JALR`

## Precondiciones

- `R10 = 0x8`
- `R11 = 0x10`
- `R31 = 0`
- `JR` cargada en la dirección `0x0`
- `JALR` cargada en la dirección `0x8`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
JR   $10
JALR $11, $31
```

Primer intento con codificación incorrecta:

```
reset
s r10 0x8
s r11 0x10
s r31 0
s [0x0] 0x0280000E
s [0x8] 0x02FE000F
s pc 0x0
r
n 2
r
```

Resultado observado en el primer intento:

```
R[ 0]: 0x0000000C
R[10]: 0x00000008
R[11]: 0x00000010
R[31]: 0x00000000
PC   : 0x00000010
```

En este primer intento, el salto sí funcionó porque el `PC` terminó en `0x00000010`, pero la dirección de retorno quedó guardada en `R0` en lugar de `R31`.

Esto mostró que hubo un error de comprensión: el registro de retorno no debía estar en el campo `rt`, sino en el campo `rd`.

Codificación corregida:

```
JR   $10       -> 0x0280000E
JALR $11, $31  -> 0x02C1F00F
```

Comandos usados para la prueba corregida:

```
reset
s r10 0x8
s r11 0x10
s r31 0
s [0x0] 0x0280000E
s [0x8] 0x02C1F00F
s pc 0x0
r
n 2
r
```

## Postcondiciones

Resultado esperado/observado después de corregir la codificación:

```
R[10]: 0x00000008
R[11]: 0x00000010
R[31]: 0x0000000C
PC   : 0x00000010
```

## Conclusiones

En el primer intento, la instrucción `JALR` no quedó codificada correctamente porque la dirección de retorno se guardó en `R0` en vez de `R31`.

El error permitió detectar que, para esta instrucción, el registro de enlace se toma del campo `rd`.

Luego de corregir la codificación, `JR` y `JALR` funcionaron correctamente.

`JR` cambió el `PC` a `0x00000008`, que era el valor guardado en `R10`.

Luego, `JALR` guardó en `R31` la dirección de retorno `0x0000000C` y cambió el `PC` a `0x00000010`, que era el valor guardado en `R11`.

El resultado confirma que `JR` realiza un salto indirecto y que `JALR` realiza un salto indirecto guardando dirección de retorno.

---

# Caso 10

## Descripción

Testeo conjunto de instrucciones tipo R de carga desde memoria con direccionamiento indexado.

En este caso se prueban `LWX`, `LHX`, `LHUX`, `LBX` y `LBUX`.

Estas instrucciones calculan la dirección efectiva usando dos registros:

```
dirección efectiva = R[rs] + R[rd]
```

Valores usados como base y offsets:

```
R10 = 0x20
R11 = 0
R12 = 4
R13 = 8
```

Valores cargados previamente en memoria:

```
[0x20] = 0x12345678
[0x24] = 0x000080FF
[0x28] = 0x00000080
```

Resultados esperados:

```
LWX:  R20 = M[R10 + R11]
LWX:  R20 = M[0x20 + 0] = M[0x20] = 0x12345678

LHX:  R21 = media palabra de M[R10 + R12] extendida con signo
LHX:  R21 = 0xFFFF80FF

LHUX: R22 = media palabra de M[R10 + R12] extendida con ceros
LHUX: R22 = 0x000080FF

LBX:  R23 = byte de M[R10 + R13] extendido con signo
LBX:  R23 = 0xFFFFFF80

LBUX: R24 = byte de M[R10 + R13] extendido con ceros
LBUX: R24 = 0x00000080
```

## Instructions

- `LWX`
- `LHX`
- `LHUX`
- `LBX`
- `LBUX`

## Precondiciones

- `R10 = 0x20`
- `R11 = 0`
- `R12 = 4`
- `R13 = 8`
- `R20 = 0`
- `R21 = 0`
- `R22 = 0`
- `R23 = 0`
- `R24 = 0`
- Datos cargados en memoria desde `0x20`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
LWX  $10, $20, $11
LHX  $10, $21, $12
LHUX $10, $22, $12
LBX  $10, $23, $13
LBUX $10, $24, $13
```

Comandos usados:

```
reset

s r10 0x20
s r11 0
s r12 4
s r13 8

s r20 0
s r21 0
s r22 0
s r23 0
s r24 0

s [0x20] 0x12345678
s [0x24] 0x000080FF
s [0x28] 0x00000080

s [0x0] 0x02A8B014
s [0x4] 0x02AAC010
s [0x8] 0x02ACC011
s [0xC] 0x02AED012
s [0x10] 0x02B0D013

s pc 0x0
r
n 5
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[10]: 0x00000020
R[11]: 0x00000000
R[12]: 0x00000004
R[13]: 0x00000008

R[20]: 0x12345678
R[21]: 0xFFFF80FF
R[22]: 0x000080FF
R[23]: 0xFFFFFF80
R[24]: 0x00000080

PC   : 0x00000014
```

## Conclusiones

Las instrucciones `LWX`, `LHX`, `LHUX`, `LBX` y `LBUX` funcionaron correctamente si los registros destino quedaron con los valores esperados.

`LWX` cargó en `R20` una palabra completa de 32 bits desde memoria, dando `0x12345678`.

`LHX` cargó en `R21` una media palabra y la extendió con signo, dando `0xFFFF80FF`.

`LHUX` cargó en `R22` una media palabra y la extendió sin signo, dando `0x000080FF`.

`LBX` cargó en `R23` un byte y lo extendió con signo, dando `0xFFFFFF80`.

`LBUX` cargó en `R24` un byte y lo extendió sin signo, dando `0x00000080`.

El `PC` quedó en `0x00000014`, lo que indica que se ejecutaron 5 instrucciones de 32 bits desde `0x0` hasta `0x10`.

---

# Caso 11

## Descripción

Testeo conjunto de instrucciones con inmediato.

En este caso se prueban `ADDI`, `LUI`, `ORI` y `XORI`.

`ADDI` suma un registro con un valor inmediato.

`LUI` carga un inmediato en la parte alta del registro.

`ORI` realiza un OR entre un registro y un inmediato.

`XORI` realiza un XOR entre un registro y un inmediato.

Valores usados:

```
R12 = 20
R13 = 0
R10 = 0
R11 = 0
```

Resultados esperados:

```
ADDI: R13 = R12 + 5
ADDI: R13 = 20 + 5 = 25 = 0x00000019

LUI:  R10 = 0x12340000

ORI:  R10 = R10 | 0x5678
ORI:  R10 = 0x12340000 | 0x00005678 = 0x12345678

XORI: R11 = R10 ^ 0x5678
XORI: R11 = 0x12345678 ^ 0x00005678 = 0x12340000
```

## Instructions

- `ADDI`
- `LUI`
- `ORI`
- `XORI`

## Precondiciones

- `R12 = 20`
- `R13 = 0`
- `R10 = 0`
- `R11 = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
ADDI $12, $13, 5
LUI  $10, 0x1234
ORI  $10, $10, 0x5678
XORI $10, $11, 0x5678
```

Comandos usados:

```
reset
s r12 20
s r13 0
s r10 0
s r11 0
s [0x0] 0x0B1A0005
s [0x4] 0x38141234
s [0x8] 0x2A945678
s [0xC] 0x32965678
s pc 0x0
r
n 4
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[10]: 0x12345678
R[11]: 0x12340000
R[12]: 0x00000014
R[13]: 0x00000019
PC   : 0x00000010
```

## Conclusiones

Las instrucciones `ADDI`, `LUI`, `ORI` y `XORI` funcionaron correctamente si los registros destino quedaron con los valores esperados.

`ADDI` guardó en `R13` el valor `25`, que en hexadecimal es `0x00000019`.

`LUI` cargó en `R10` la parte alta `0x1234`, dejando el valor `0x12340000`.

`ORI` completó la parte baja de `R10`, dejando el valor `0x12345678`.

`XORI` guardó en `R11` el resultado de aplicar XOR entre `0x12345678` y `0x00005678`, dando `0x12340000`.

El `PC` quedó en `0x00000010`, lo que indica que se ejecutaron 4 instrucciones de 32 bits desde `0x0` hasta `0xC`.

---

# Caso 12

## Descripción

Testeo conjunto de instrucciones lógicas con inmediato de formato L.

En este caso se prueban `ANDI`, `ANDIH`, `ORIH` y `XORIH`.

Estas instrucciones realizan operaciones lógicas entre un registro y un inmediato.

La diferencia entre la versión normal y la versión con `H` es que la versión con `H` aplica el inmediato en la parte alta del valor:

```
h = 0 -> inmediato en parte baja: 0x000000FF
h = 1 -> inmediato en parte alta: 0x00FF0000
```

Valores usados:

```
R10 = 0x12345678
R11 = 0
R12 = 0
R13 = 0
R14 = 0
```

Resultados esperados:

```
ANDI:  R11 = R10 & 0x000000FF
ANDI:  R11 = 0x12345678 & 0x000000FF = 0x00000078

ANDIH: R12 = R10 & 0x00FF0000
ANDIH: R12 = 0x12345678 & 0x00FF0000 = 0x00340000

ORIH:  R13 = R10 | 0x00FF0000
ORIH:  R13 = 0x12345678 | 0x00FF0000 = 0x12FF5678

XORIH: R14 = R10 ^ 0x00FF0000
XORIH: R14 = 0x12345678 ^ 0x00FF0000 = 0x12CB5678
```

## Instructions

- `ANDI`
- `ANDIH`
- `ORIH`
- `XORIH`

## Precondiciones

- `R10 = 0x12345678`
- `R11 = 0`
- `R12 = 0`
- `R13 = 0`
- `R14 = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
ANDI  $10, $11, 0x00FF
ANDIH $10, $12, 0x00FF
ORIH  $10, $13, 0x00FF
XORIH $10, $14, 0x00FF
```

Comandos usados:

```
reset
s r10 0x12345678
s r11 0
s r12 0
s r13 0
s r14 0
s [0x0] 0x229600FF
s [0x4] 0x229900FF
s [0x8] 0x2A9B00FF
s [0xC] 0x329D00FF
s pc 0x0
r
n 4
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[10]: 0x12345678
R[11]: 0x00000078
R[12]: 0x00340000
R[13]: 0x12FF5678
R[14]: 0x12CB5678
PC   : 0x00000010
```

## Conclusiones

Las instrucciones `ANDI`, `ANDIH`, `ORIH` y `XORIH` funcionaron correctamente si los registros destino quedaron con los valores esperados.

`ANDI` guardó en `R11` el resultado de aplicar AND con el inmediato en la parte baja, dando `0x00000078`.

`ANDIH` guardó en `R12` el resultado de aplicar AND con el inmediato en la parte alta, dando `0x00340000`.

`ORIH` guardó en `R13` el resultado de aplicar OR con el inmediato en la parte alta, dando `0x12FF5678`.

`XORIH` guardó en `R14` el resultado de aplicar XOR con el inmediato en la parte alta, dando `0x12CB5678`.

El `PC` quedó en `0x00000010`, lo que indica que se ejecutaron 4 instrucciones de 32 bits desde `0x0` hasta `0xC`.

En caso de que `ANDI` o `ANDIH` no coincidan con el resultado esperado, el caso puede quedar registrado como posible fallo relacionado con `ANDI`, ya que en el manual se menciona que existe un bug importante en esa instrucción.


---

# Caso 13

## Descripción

Testeo conjunto de instrucciones de carga y guardado de palabra en memoria.

En este caso se prueban `LW` y `SW`.

`LW` carga una palabra de 32 bits desde memoria hacia un registro.

`SW` guarda una palabra de 32 bits desde un registro hacia memoria.

Ambas usan una dirección efectiva calculada con un registro base y un inmediato:

```
dirección efectiva = R[rs] + inmediato
```

Valores usados:

```
R10 = 0x20
R11 = 0
R12 = 0xDEADBEEF
M[0x20] = 0x12345678
M[0x24] = 0
```

Resultados esperados:

```
LW: R11 = M[R10 + 0]
LW: R11 = M[0x20] = 0x12345678

SW: M[R10 + 4] = R12
SW: M[0x24] = 0xDEADBEEF
```

## Instructions

- `LW`
- `SW`

## Precondiciones

- `R10 = 0x20`
- `R11 = 0`
- `R12 = 0xDEADBEEF`
- `M[0x20] = 0x12345678`
- `M[0x24] = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
LW $10, $11, 0
SW $10, $12, 4
```

Comandos usados:

```
reset
s r10 0x20
s r11 0
s r12 0xDEADBEEF
s [0x20] 0x12345678
s [0x24] 0
s [0x0] 0x42960000
s [0x4] 0x4A980004
s pc 0x0
r
n 2
r
x xw 0x20 2
```

## Postcondiciones

Resultado esperado/observado en registros:

```
R[10]: 0x00000020
R[11]: 0x12345678
R[12]: 0xDEADBEEF
PC   : 0x00000008
```

Resultado esperado/observado en memoria:

```
M[0x20]: 0x12345678
M[0x24]: 0xDEADBEEF
```

## Conclusiones

Las instrucciones `LW` y `SW` funcionaron correctamente si los registros y la memoria quedaron con los valores esperados.

`LW` cargó en `R11` la palabra almacenada en `M[0x20]`, dando `0x12345678`.

`SW` guardó el valor de `R12` en `M[0x24]`, dejando en memoria el valor `0xDEADBEEF`.

El `PC` quedó en `0x00000008`, lo que indica que se ejecutaron 2 instrucciones de 32 bits desde `0x0` y `0x4`.


---

# Caso 14

## Descripción

Testeo conjunto de instrucciones de carga de media palabra desde memoria.

En este caso se prueban `LH` y `LHU`.

`LH` carga 16 bits desde memoria y los extiende con signo.

`LHU` carga 16 bits desde memoria y los extiende sin signo.

Ambas usan una dirección efectiva calculada con un registro base y un inmediato:

```
dirección efectiva = R[rs] + inmediato
```

Valores usados:

```
R10 = 0x20
R11 = 0
R12 = 0
M[0x20] contiene en su media palabra baja el valor 0x80FF
```

Resultados esperados:

```
LH:  R11 = media palabra baja de M[R10 + 0] extendida con signo
LH:  R11 = 0xFFFF80FF

LHU: R12 = media palabra baja de M[R10 + 0] extendida sin signo
LHU: R12 = 0x000080FF
```

## Instructions

- `LH`
- `LHU`

## Precondiciones

- `R10 = 0x20`
- `R11 = 0`
- `R12 = 0`
- En memoria, la media palabra baja de `M[0x20]` contiene `0x80FF`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
LH  $10, $11, 0
LHU $10, $12, 0
```

Comandos usados:

```
reset
s r10 0x20
s r11 0
s r12 0
s [0x20] 0x000080FF
s [0x0] 0x62960000
s [0x4] 0x6A980000
s pc 0x0
r
n 2
r
x xw 0x20
```

## Postcondiciones

Resultado observado:

```
R[10]: 0x00000020
R[11]: 0xFFFF80FF
R[12]: 0x000080FF
PC   : 0x00000008
```

También se observó que la última operación de memoria fue una lectura desde `0x00000020`:

```
Last Memory Operation:
Address: 0x00000020 | Size: 0x00000002 | Type: READ
```

## Conclusiones

Las instrucciones `LH` y `LHU` funcionaron correctamente porque los registros destino quedaron con los valores esperados.

`LH` cargó la media palabra `0x80FF` y la extendió con signo, dando `0xFFFF80FF`.

`LHU` cargó la media palabra `0x80FF` y la extendió sin signo, dando `0x000080FF`.

Aunque en memoria podían quedar valores de casos anteriores, la prueba sigue siendo válida porque estas instrucciones leen solamente 16 bits desde la dirección efectiva.

---

# Caso 15

## Descripción

Testeo conjunto de instrucciones de carga de byte desde memoria.

En este caso se prueban `LB` y `LBU`.

`LB` carga un byte desde memoria y lo extiende con signo.

`LBU` carga un byte desde memoria y lo extiende sin signo.

Ambas usan una dirección efectiva calculada con un registro base y un inmediato:

```
dirección efectiva = R[rs] + inmediato
```

Valores usados:

```
R10 = 0x40
R11 = 0
R12 = 0
M[0x40] = 0x00000080
```

Resultados esperados:

```
LB:  R11 = byte de M[R10 + 0] extendido con signo
LB:  R11 = 0xFFFFFF80

LBU: R12 = byte de M[R10 + 0] extendido sin signo
LBU: R12 = 0x00000080
```

## Instructions

- `LB`
- `LBU`

## Precondiciones

- `R10 = 0x40`
- `R11 = 0`
- `R12 = 0`
- `M[0x40] = 0x00000080`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
LB  $10, $11, 0
LBU $10, $12, 0
```

Comandos usados:

```
reset
s r10 0x40
s r11 0
s r12 0
s [0x40] 0x00000080
s [0x0] 0x72960000
s [0x4] 0x7A980000
s pc 0x0
r
n 2
r
x xw 0x40
```

## Postcondiciones

Resultado esperado/observado:

```
R[10]: 0x00000040
R[11]: 0xFFFFFF80
R[12]: 0x00000080
PC   : 0x00000008
```

Valor en memoria usado para la prueba:

```
M[0x40]: 0x00000080
```

## Conclusiones

Las instrucciones `LB` y `LBU` funcionaron correctamente si los registros destino quedaron con los valores esperados.

`LB` cargó el byte `0x80` y lo extendió con signo, dando `0xFFFFFF80`.

`LBU` cargó el byte `0x80` y lo extendió sin signo, dando `0x00000080`.

El `PC` quedó en `0x00000008`, lo que indica que se ejecutaron 2 instrucciones de 32 bits desde `0x0` y `0x4`.

---

# Caso 16

## Descripción

Testeo conjunto de instrucciones de guardado parcial en memoria.

En este caso se prueban `SH` y `SB`.

`SH` guarda los 16 bits bajos de un registro en memoria.

`SB` guarda los 8 bits bajos de un registro en memoria.

Ambas usan una dirección efectiva calculada con un registro base y un inmediato:

```
dirección efectiva = R[rs] + inmediato
```

Valores usados:

```
R10 = 0x60
R11 = 0x1234BEEF
R12 = 0xABCDEF80
M[0x60] = 0
M[0x64] = 0
```

Resultados esperados:

```
SH: M[R10 + 0] = R11[15:0]
SH: M[0x60] = 0xBEEF

SB: M[R10 + 4] = R12[7:0]
SB: M[0x64] = 0x80
```

## Instructions

- `SH`
- `SB`

## Precondiciones

- `R10 = 0x60`
- `R11 = 0x1234BEEF`
- `R12 = 0xABCDEF80`
- `M[0x60] = 0`
- `M[0x64] = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
SH $10, $11, 0
SB $10, $12, 4
```

Comandos usados:

```
reset
s r10 0x60
s r11 0x1234BEEF
s r12 0xABCDEF80
s [0x60] 0
s [0x64] 0
s [0x0] 0x52960000
s [0x4] 0x5A980004
s pc 0x0
r
n 2
r
x xw 0x60 2
```

## Postcondiciones

Resultado esperado/observado en registros:

```
R[10]: 0x00000060
R[11]: 0x1234BEEF
R[12]: 0xABCDEF80
PC   : 0x00000008
```

Resultado esperado/observado en memoria:

```
M[0x60]: 0x0000BEEF
M[0x64]: 0x00000080
```

## Conclusiones

Las instrucciones `SH` y `SB` funcionaron correctamente si la memoria quedó con los valores esperados.

`SH` guardó en `M[0x60]` los 16 bits bajos de `R11`, es decir `0xBEEF`.

`SB` guardó en `M[0x64]` los 8 bits bajos de `R12`, es decir `0x80`.

El `PC` quedó en `0x00000008`, lo que indica que se ejecutaron 2 instrucciones de 32 bits desde `0x0` y `0x4`.

---

# Caso 17

## Descripción

Testeo conjunto de instrucciones de comparación con inmediato.

En este caso se prueban `SLTI` y `SLTIU`.

`SLTI` compara un registro contra un inmediato con signo.

`SLTIU` compara un registro contra un inmediato sin signo.

Valores usados:

```
R10 = 0xFFFFFFFF
inmediato = 1
```

Estos valores permiten diferenciar comparación con signo y sin signo:

```
0xFFFFFFFF como signed   = -1
0xFFFFFFFF como unsigned = 4294967295
```

Resultados esperados:

```
SLTI:  R11 = (R10 < 1) con signo
SLTI:  R11 = (-1 < 1) = 1

SLTIU: R12 = (R10 < 1) sin signo
SLTIU: R12 = (4294967295 < 1) = 0
```

## Instructions

- `SLTI`
- `SLTIU`

## Precondiciones

- `R10 = 0xFFFFFFFF`
- `R11 = 0`
- `R12 = 0`
- Instrucciones cargadas desde la dirección `0x0`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
SLTI  $10, $11, 1
SLTIU $10, $12, 1
```

Comandos usados:

```
reset
s r10 0xFFFFFFFF
s r11 0
s r12 0
s [0x0] 0xB2960001
s [0x4] 0xBA980001
s pc 0x0
r
n 2
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[10]: 0xFFFFFFFF
R[11]: 0x00000001
R[12]: 0x00000000
PC   : 0x00000008
```

## Conclusiones

Las instrucciones `SLTI` y `SLTIU` funcionaron correctamente si los registros destino quedaron con los valores esperados.

`SLTI` guardó en `R11` el valor `1`, porque interpreta `0xFFFFFFFF` como `-1`, y `-1 < 1` es verdadero.

`SLTIU` guardó en `R12` el valor `0`, porque interpreta `0xFFFFFFFF` como `4294967295`, y `4294967295 < 1` es falso.

El `PC` quedó en `0x00000008`, lo que indica que se ejecutaron 2 instrucciones de 32 bits desde `0x0` y `0x4`.

---

# Caso 18

## Descripción

Testeo de instrucciones de salto condicional.

En este caso se prueban por separado `BEQ`, `BNE`, `BLT`, `BGT`, `BLE` y `BGE`.

Estas instrucciones modifican el flujo del programa si se cumple una condición particular entre dos registros.

La forma de comprobarlas fue la misma en todos los casos: después de cada branch se colocó una instrucción `ADDI` que sumaba `1` a `R20`.

Si el branch funcionaba correctamente, debía saltear esa instrucción. Por lo tanto, `R20` debía quedar en `0`.

Valores usados:

```
R10 = 5
R11 = 5
R12 = 6
R20 = 0
```

Condiciones probadas:

```
BEQ: R10 == R11  -> 5 == 5  -> debe saltar
BNE: R10 != R12  -> 5 != 6  -> debe saltar
BLT: R10 < R12   -> 5 < 6   -> debe saltar
BGT: R12 > R10   -> 6 > 5   -> debe saltar
BLE: R10 <= R11  -> 5 <= 5  -> debe saltar
BGE: R12 >= R10  -> 6 >= 5  -> debe saltar
```

En todos los casos se verificó que `R20` quedara en `0`, demostrando que cada branch salteó correctamente la instrucción `ADDI`.

## Instructions

- `BEQ`
- `BNE`
- `BLT`
- `BGT`
- `BLE`
- `BGE`

## Precondiciones

- `R10 = 5`
- `R11 = 5`
- `R12 = 6`
- `R20 = 0`
- Cada branch fue probado por separado.
- En cada prueba, después del branch se cargó una instrucción `ADDI R20, R20, 1`.
- `PC = 0x0` al inicio de cada prueba.

## Code

Pseudo-estructura usada en cada prueba:

```
BRANCH condición, offset 1
ADDI R20, R20, 1
```

Si el branch funciona correctamente, salta la instrucción `ADDI` y `R20` queda en `0`.

Pruebas realizadas:

```
BEQ R10, R11, 1
ADDI R20, R20, 1

BNE R10, R12, 1
ADDI R20, R20, 1

BLT R10, R12, 1
ADDI R20, R20, 1

BGT R12, R10, 1
ADDI R20, R20, 1

BLE R10, R11, 1
ADDI R20, R20, 1

BGE R12, R10, 1
ADDI R20, R20, 1
```

Comandos usados para cada prueba, cambiando solamente la instrucción branch en `0x0` (para todos los casos de branch):

```
reset
s r10 5
s r11 5
s r12 6
s r20 0
s [0x0] <codigo_branch_especial>
s [0x4] 0x0D280001
s pc 0x0
r
n 1
r
```

Códigos usados para cada branch:

```
BEQ R10, R11, 1 -> 0x82960001
BNE R10, R12, 1 -> 0x8A980001
BLT R10, R12, 1 -> 0x92980001
BGT R12, R10, 1 -> 0x9B140001
BLE R10, R11, 1 -> 0xA2960001
BGE R12, R10, 1 -> 0xAB140001
```

## Postcondiciones

Resultado esperado/observado en cada prueba:

```
R[20]: 0x00000000
PC   : 0x00000008
```

Resultados por instrucción:

```
BEQ: R20 quedó en 0, por lo tanto salteó correctamente el ADDI.
BNE: R20 quedó en 0, por lo tanto salteó correctamente el ADDI.
BLT: R20 quedó en 0, por lo tanto salteó correctamente el ADDI.
BGT: R20 quedó en 0, por lo tanto salteó correctamente el ADDI.
BLE: R20 quedó en 0, por lo tanto salteó correctamente el ADDI.
BGE: R20 quedó en 0, por lo tanto salteó correctamente el ADDI.
```

## Conclusiones

Las instrucciones `BEQ`, `BNE`, `BLT`, `BGT`, `BLE` y `BGE` funcionaron correctamente en las pruebas realizadas.

Cada branch fue probado con una condición distinta, correspondiente a su funcionamiento particular.

En todos los casos, `R20` quedó en `0`, lo que demuestra que el salto se realizó correctamente y que la instrucción `ADDI R20, R20, 1` fue salteada.

Por lo tanto, se concluye que los saltos condicionales evaluaron correctamente sus condiciones y modificaron el `PC` como se esperaba.

---

# Caso 19

## Descripción

Testeo conjunto de instrucciones de salto incondicional.

En este caso se prueban `J` y `JAL`.

`J` realiza un salto incondicional a una dirección.

`JAL` realiza un salto incondicional y además guarda la dirección de retorno en `R31`.

Valores usados:

```
R20 = 0
R31 = 0
```

La idea del test es colocar una instrucción `ADDI R20, R20, 1` después de cada salto.

Si el salto funciona correctamente, esa instrucción debe ser salteada y `R20` debe quedar en `0`.

Además, en el caso de `JAL`, se espera que guarde en `R31` la dirección de retorno.

Resultados esperados:

```
J:   salta desde 0x0 hasta 0x8
J:   saltea la instrucción ubicada en 0x4

JAL: salta desde 0x8 hasta 0x10
JAL: guarda en R31 la dirección PC + 4
JAL: R31 = 0x8 + 4 = 0xC
JAL: saltea la instrucción ubicada en 0xC
```

Por lo tanto:

```
R20 = 0
R31 = 0x0000000C
PC = 0x00000010
```

## Instructions

- `J`
- `JAL`

## Precondiciones

- `R20 = 0`
- `R31 = 0`
- `J` cargada en la dirección `0x0`
- Una instrucción `ADDI R20, R20, 1` cargada en `0x4`
- `JAL` cargada en la dirección `0x8`
- Otra instrucción `ADDI R20, R20, 1` cargada en `0xC`
- `PC = 0x0`

## Code

Pseudo-instrucciones:

```
J 0x8
ADDI R20, R20, 1

JAL 0x10
ADDI R20, R20, 1
```

Comandos usados:

```
reset
s r20 0
s r31 0
s [0x0] 0x10000002
s [0x4] 0x0D280001
s [0x8] 0x18000004
s [0xC] 0x0D280001
s pc 0x0
r
n 2
r
```

## Postcondiciones

Resultado esperado/observado:

```
R[20]: 0x00000000
R[31]: 0x0000000C
PC   : 0x00000010
```

## Conclusiones

Las instrucciones `J` y `JAL` funcionaron correctamente si los registros y el `PC` quedaron con los valores esperados.

`J` saltó desde `0x0` hasta `0x8`, salteando la instrucción `ADDI` ubicada en `0x4`.

`JAL` saltó desde `0x8` hasta `0x10`, salteando la instrucción `ADDI` ubicada en `0xC`.

Además, `JAL` guardó en `R31` la dirección de retorno `0x0000000C`.

Como `R20` quedó en `0`, se confirma que ambas instrucciones `ADDI` fueron salteadas correctamente.

El `PC` quedó en `0x00000010`, lo que indica que el flujo del programa terminó en la dirección esperada después de ejecutar los dos saltos.

---

# Aclaración final sobre instrucciones no completadas y fallidas

Durante las pruebas se intentó cubrir la mayor cantidad posible de instrucciones de la CPU RTM32/STX4.

Las instrucciones `TRAP` y `RFT` no fueron testeadas en estos casos porque dependen del mecanismo de excepciones de la CPU y porque segun entendi no iban a pode probarse.

Por ese motivo, ambas instrucciones requieren preparar un contexto especial de excepción/interrupción, y se dejaron fuera de estas pruebas básicas.

También se probaron `CFS` y `CTS`, pero el resultado fue inconcluso. El `PC` avanzó correctamente, lo que indica que las instrucciones fueron buscadas y ejecutadas, pero no se observaron los cambios esperados en los registros especiales. Por lo tanto, quedaron registradas como prueba fallida o no concluyente.

En resumen:

```
Instrucciones testeadas correctamente:
ADD, SUB, AND, OR, XOR, NOR,
SLL, SRL, SRA,
SLLR, SRLR, SRAR,
SLT, SLTU,
JR, JALR,
MUL, MULH, MULHU,
DIV, DIVU, REST, RESTU,
LWX, LHX, LHUX, LBX, LBUX,
ADDI, LUI, ORI, XORI,
ANDI, ANDIH, ORIH, XORIH,
LW, SW,
LH, LHU,
LB, LBU,
SH, SB,
SLTI, SLTIU,
BEQ, BNE, BLT, BGT, BLE, BGE,
J, JAL.

Instrucciones probadas pero inconclusas:
CFS, CTS.

Instrucciones no testeadas:
TRAP, RFT.
```
:)

░░░░░░░░░░░░▄▄░░░░░░░░░░░░░░
░░░░░░░░░░░█░░█░░░░░░░░░░░░░
░░░░░░░░░░░█░░█░░░░░░░░░░░░░
░░░░░░░░░░█░░░█░░░░░░░░░░░░░
░░░░░░░░░█░░░░█░░░░░░░░░░░░░
██████▄▄█░░░░░██████▄░░░░░░░
▓▓▓▓▓▓█░░░░░░░░░░░░░░█░░░░░░
▓▓▓▓▓▓█░░░░░░░░░░░░░░█░░░░░░
▓▓▓▓▓▓█░░░░░░░░░░░░░░█░░░░░░
▓▓▓▓▓▓█░░░░░░░░░░░░░░█░░░░░░
▓▓▓▓▓▓█░░░░░░░░░░░░░░█░░░░░░
▓▓▓▓▓▓█████░░░░░░░░░██░░░░░░
█████▀░░░░▀▀████████░░░░░░░░
░░░░░░░░░░░░░░░░░░░░░░░░░░░░


