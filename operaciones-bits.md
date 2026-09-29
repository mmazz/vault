# Operaciones de bits en C

Para qué en el proyecto:
- campos de bits de los registros de configuración de la IMU (1.5);
- armar valores de 16 bits con signo a partir de dos bytes (1.5);
- CRC y contador de secuencia en el transporte (1.7);
- bit flips en los coeficientes de calibración (fase 9).

---

## 1. Notación

- **Numeración:** el bit 0 es el menos significativo (LSB). En las hojas de datos, `[4:3]` es el campo formado por los bits 4 y 3.
- **Hexadecimal:** cada dígito son 4 bits. `0x18` = `0001 1000`. Es la notación de las hojas de datos y del código.
  - `0b…` (binario) es una extensión de GCC; es estándar recién en C23.
- **Tipos:** usar los de ancho fijo de `<stdint.h>` (`uint8_t`, `uint16_t`, `int16_t`…). El ancho de `int` depende de la plataforma: es de 32 bits en el ESP32 y en el Cortex-M7, pero no en general.
- **Regla práctica:** operar bits sobre tipos **sin signo**. Con signo aparecen los casos indefinidos de las secciones 6 y 7.

---

## 2. Operadores

| C | Nombre | Bit resultado | Uso con máscara | Ejemplo (4 bits) |
|---|---|---|---|---|
| `&` | AND | 1 si los dos son 1 | conservar o consultar bits | `1101 & 0011 = 0001` |
| `\|` | OR | 1 si alguno es 1 | poner bits en 1 | `1100 \| 0011 = 1111` |
| `^` | XOR | 1 si son distintos | invertir bits | `1101 ^ 0011 = 1110` |
| `~` | NOT | invierte todos | armar la máscara para borrar | `~1101 = 0010` |
| `<<` | shift a izquierda | entran ceros por la derecha | ubicar un campo en su posición | `0011 << 1 = 0110` |
| `>>` | shift a derecha | ver sección 6 | bajar un campo a la posición 0 | `1100 >> 1 = 0110` |

- El resultado de `~` depende del ancho: `~1101` es `0010` en 4 bits, pero en C se calcula al ancho de `int` (sección 7).
- No confundir con los operadores lógicos `&&`, `||`, `!`: tratan el valor entero como verdadero o falso y dan 0 o 1.

---

## 3. Patrones con una máscara

Ejemplo con `x = 1101` y `mask = 0011`:

| Qué | Expresión | Resultado |
|---|---|---|
| Consultar | `x & mask` (≠ 0 si alguno de esos bits está en 1) | `0001` |
| Poner en 1 | `x \|= mask` | `1111` |
| Poner en 0 | `x &= ~mask` | `1100` |
| Invertir | `x ^= mask` | `1110` |

Construir máscaras:

| Máscara | Expresión | Ejemplo |
|---|---|---|
| Solo el bit `n` | `1u << n` | `n = 2` → `0100` |
| Los `k` bits más bajos | `(1u << k) - 1` | `k = 3` → `0111` |

---

## 4. Campos de varios bits

Un registro de configuración suele tener varios campos. Para un campo de `ancho` bits que empieza en el bit `pos`, con `m = (1u << ancho) - 1`:

| Qué | Expresión |
|---|---|
| Leer el campo | `(x >> pos) & m` |
| Escribir `v` en el campo sin tocar el resto | `x = (x & ~(m << pos)) \| ((v & m) << pos)` |

La segunda es un **read-modify-write**: leer el registro, cambiar solo el campo y escribir el byte completo. Se usa porque los demás bits del registro tienen otras funciones. Algunos son reservados, y la hoja de datos dice qué valor hay que escribir en ellos.

---

## 5. XOR y los bit flips

| Propiedad | Consecuencia |
|---|---|
| `x ^ 0 = x` | los bits con máscara 0 no cambian |
| `x ^ x = 0` | un valor XOR consigo mismo se anula |
| `x ^ m ^ m = x` | aplicar la misma máscara dos veces deshace el cambio |
| conmutativa y asociativa | el orden de los XOR no importa |

- **Un bit flip en el bit `n` es `x ^ (1u << n)`.** Una falla transitoria que invierte `k` bits es un XOR con una máscara que tiene `k` unos.
- **Distancia de Hamming** entre `a` y `b`: cantidad de unos en `a ^ b`.
- **Paridad:** XOR de todos los bits.

Esto es la base de la paridad, del CRC (1.7) y del ECC (fase 9).

---

## 6. Desplazamientos: cuándo equivalen a multiplicar o dividir

**Sin signo:** está todo definido.
- `x << n` = x · 2ⁿ **módulo 2^ancho**: los bits que salen por la izquierda se pierden.
- `x >> n` = ⌊x / 2ⁿ⌋, **exacto** (no "aproximadamente"): entran ceros por la izquierda.

**Con signo:**
- `<<` de un valor negativo, o que desborda el tipo: **comportamiento indefinido**.
- `>>` de un valor negativo: **definido por la implementación**. GCC hace un desplazamiento aritmético: replica el bit de signo, así que redondea hacia −∞. La división `/` de C trunca hacia 0: con negativos, los dos resultados difieren.

**En cualquier tipo:** desplazar una cantidad mayor o igual al ancho del tipo (después de la promoción de la sección 7) es indefinido.

No reemplazar multiplicaciones por shifts "por eficiencia": el compilador ya lo hace solo. Se usa shift cuando la operación es sobre bits.

---

## 7. Trampas de C

1. **Precedencia.** `x & mask == 0` se lee `x & (mask == 0)`, porque `==` tiene más precedencia que `&`. Lo mismo con `a << 8 + b`, que es `a << (8 + b)`. Con operadores de bits: paréntesis siempre.
2. **Promoción entera.** Antes de operar, `uint8_t` y `uint16_t` se convierten a `int`. Por eso `~(uint8_t)0x03` vale `0xFFFFFFFC` (un `int`) y no `0xFC`. Guardado en un `uint8_t` se trunca bien, pero comparado directamente con otro `uint8_t` da falso.
3. **Literales con signo.** `1` es un `int` con signo: `1 << 31` desplaza hasta el bit de signo y es indefinido. `1u << 31` está bien.
4. **`char` sin especificar** puede ser con o sin signo según la plataforma. Para bytes: `uint8_t`.

---

## 8. Cómo se representan los enteros

### 8.1 Sin signo

valor = Σ bᵢ · 2ⁱ. Con n bits, el rango es 0 … 2ⁿ − 1.

### 8.2 Complemento a dos

El bit más significativo pesa **negativo**:

valor = −bₙ₋₁ · 2ⁿ⁻¹ + Σᵢ₌₀ⁿ⁻² bᵢ · 2ⁱ

- **Rango:** −2ⁿ⁻¹ … 2ⁿ⁻¹ − 1. Para 16 bits, −32768 … 32767: hay un negativo más que positivos.
- **Negar:** −x = `~x + 1`.
- **Los bits no tienen tipo; el tipo decide cómo se leen.** `0xFF` vale 255 como `uint8_t` y −1 como `int8_t`.
- **Extensión de signo:** al pasar un negativo a un tipo más ancho, el bit de signo se copia en los bits nuevos. −1 en 8 bits (`0xFF`) es `0xFFFF` en 16 bits.
- **En C:** `int16_t` y los demás `intN_t` están garantizados en complemento a dos desde C99. Desde C23, lo están todos los enteros con signo.

### 8.3 Endianness

Es el orden de los bytes de un valor de varios bytes.

| | Primero (dirección más baja, o primero en el bus) |
|---|---|
| Big-endian | el byte más significativo |
| Little-endian | el byte menos significativo |

- El ESP32 (Xtensa) y el Cortex-M7 de la STM32 trabajan en little-endian.
- El orden en que la IMU entrega cada valor de 16 bits está en el mapa de registros (RM-MPU-6500A-00, registros `*_OUT_H` y `*_OUT_L`).
- Si el sensor y el MCU usan órdenes distintos, los bytes no se pueden copiar tal cual a un entero.

---

## 9. Preguntas para 1.5 (sin mirar)

1. Tenés dos bytes, `hi` y `lo`, que forman un entero de 16 bits en complemento a dos, big-endian. ¿Qué operaciones y qué tipos usás para obtener el valor con signo correcto? ¿Dónde puede fallar por la promoción entera o por el signo?
2. ¿Por qué no alcanza con copiar los 14 bytes de la ráfaga, con `memcpy`, a un arreglo de `int16_t`?
3. Para escribir un campo de 2 bits de un registro de configuración, ¿escribís el byte entero o hacés read-modify-write? ¿Qué dice la hoja sobre los bits reservados de ese registro?
4. En GCC, ¿cuánto dan `-7 >> 1` y `-7 / 2`? ¿Por qué son distintos?
5. ¿Cómo contás cuántos bits cambiaron entre dos valores?

---

## Fuentes

- Kernighan y Ritchie, *The C Programming Language*, 2.ª ed.: §2.7 (conversiones de tipo), §2.9 (operadores de bits), §2.12 (precedencia).
- Estándar C11, borrador N1570: §6.3.1.1 (promoción entera), §6.5.7 (desplazamientos), §7.20.1.1 (tipos `intN_t`).
- Manual de GCC, sección "Integers" de *C Implementation-Defined Behavior* (`>>` sobre negativos).
- InvenSense RM-MPU-6500A-00 rev. 2.1: registros de salida del acelerómetro, la temperatura y el giróscopo.
