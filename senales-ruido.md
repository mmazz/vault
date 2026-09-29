# Señales y ruido

Cómo una magnitud física se convierte en una secuencia de números, qué se pierde en el camino, y cómo se caracteriza el ruido de un sensor. La mecánica de bits (complemento a dos, endianness) está en [[operaciones-bits]]; las distribuciones, en [[probabilidad-distribuciones]].

---

## 0. La idea que ordena todo

Medir con un sistema digital es hacer dos discretizaciones: en **amplitud** (el ADC redondea a un paso) y en **tiempo** (se toma una muestra cada $1/f_s$). Cada una tiene su error y su condición de validez. Después, el ruido que queda se describe según **cómo se reparte en frecuencia**, y eso decide si promediar sirve.

| Efecto | Causa | Sección |
|---|---|---|
| Error de redondeo del valor | paso del ADC o del formato numérico | 1 |
| Frecuencias que aparecen donde no están | muestrear sin filtrar antes | 2 |
| Incerteza en los instantes de muestreo | jitter del reloj, resolución del instrumento | 3 |
| Cuánto ruido hay y a qué frecuencias | espectro, PSD | 4 |
| Si promediar sirve y hasta cuándo | tipo de ruido, varianza de Allan | 5, 6 |
| Cambiar ruido por ancho de banda | filtros digitales | 7 |

---

## 1. De la magnitud al número

### 1.1 Escala, resolución y exactitud

- **Escala:** un ADC de $n$ bits con rango $\pm R$ tiene un paso (1 LSB) de $\Delta = 2R/2^n$. En la MPU6050 a ±2 g: 16384 LSB/g, o sea $\Delta \approx 61$ µg.
- **Resolución:** el cambio más chico que el sistema puede indicar.
- **Exactitud:** cuán cerca está la indicación del valor verdadero. Depende de offset, ganancia, no linealidad y temperatura, no del número de bits.

En la MPU6050 (hoja de datos, §6.2): tolerancia de fábrica del cero de ±50 mg en X e Y y de ±80 mg en Z; ±3 % en la sensibilidad. La resolución es ~1000 veces mejor que la exactitud sin calibrar.

> Muchos bits no hacen exacto a un sensor. La resolución dice cuánto se puede **distinguir**; la exactitud, cuánto se puede **creer**.

### 1.2 Error de cuantización

Redondear al nivel más cercano deja un error en $[-\Delta/2, \Delta/2]$. Si ese error se reparte uniforme:

$$
\sigma_q = \frac{\Delta}{\sqrt{12}} \approx 0{,}29\,\Delta
$$

→ [[probabilidad-distribuciones#Error de cuantización]].

**Condición de validez.** El modelo uniforme supone que la señal "barre" muchos pasos, o que tiene un ruido propio comparable al paso.
- Si la entrada es constante y su ruido es mucho menor que Δ, casi todas las muestras redondean al mismo nivel. El error es un **sesgo** fijo, no un ruido, y promediar no lo reduce.
- Con ruido de σ ≳ Δ/2, el modelo uniforme ya funciona bien. El ruido hace de ***dither***: aleatoriza el error de redondeo.
- Si el ruido del sensor es muchos LSB, la cuantización es despreciable: las varianzas se suman y $\sqrt{\sigma_\text{ruido}^2 + \Delta^2/12} \approx \sigma_\text{ruido}$.

### 1.3 Punto flotante

| | float32 | float64 |
|---|---|---|
| Bits de mantisa | 23 (+1 implícito) | 52 (+1 implícito) |
| ε (distancia de 1 al siguiente) | $2^{-23} \approx 1{,}2\times10^{-7}$ | $2^{-52} \approx 2{,}2\times10^{-16}$ |
| Cifras significativas | ~7 | ~16 |

- El espaciado es **relativo**: cerca de un valor $x$ es $\approx \varepsilon\,\lvert x\rvert$.
- **Cancelación catastrófica:** restar dos números grandes y casi iguales deja pocas cifras correctas. Por eso $E[X^2] - \mu^2$ es mala receta numérica cuando $\mu \gg \sigma$: conviene restar la media primero (dos pasadas) o usar el algoritmo de Welford.
- **Timestamps:** en float32, cerca de $t = 1000$ s el paso es $2^{-14}$ s ≈ 61 µs, mayor que el jitter de un reloj decente. Los instantes se guardan como enteros de *ticks* o en float64.
- **Punto fijo:** un entero con una escala implícita. Se elige la escala para que el rango entre en el tipo y el paso sea menor que la incerteza que se quiere mostrar.

---

## 2. Muestreo y aliasing

### 2.1 Nyquist y plegado

Muestreando a $f_s$ solo se puede representar sin ambigüedad el intervalo $[0, f_s/2]$ (frecuencia de Nyquist). Una componente de frecuencia $f$ aparece en

$$
f_\text{alias} = \big\lvert f - f_s\cdot\operatorname{round}(f/f_s)\big\rvert
$$

El espectro se "pliega" alrededor de los múltiplos de $f_s/2$. Por ejemplo, 130 Hz muestreados a 100 Hz aparecen en 30 Hz.

### 2.2 El antialiasing va antes

Después de muestrear, un alias **es** una señal de esa frecuencia: no hay forma de distinguirlo de una verdadera. Un filtro digital aplicado después no lo saca. El filtro antialiasing tiene que actuar **antes** del muestreo, o antes de la decimación que produce el plegado.

**Decimación:** bajar la frecuencia de muestreo quedándose con 1 de cada $M$ muestras. Es un nuevo muestreo, con un nuevo Nyquist en $f_s/(2M)$: primero se filtra, después se descarta.

Un filtro real no es un escalón. Lo que está un poco por encima de su frecuencia de corte entra atenuado solo parcialmente, y se pliega igual.

### 2.3 La cadena de muestreo de la MPU6050

```
Sample Rate = Gyroscope Output Rate / (1 + SMPLRT_DIV)
```

- **Gyroscope Output Rate:** 8 kHz con el filtro digital deshabilitado (`DLPF_CFG` = 0 o 7); 1 kHz en los demás casos.
- **El acelerómetro sale siempre a 1 kHz.** Con Sample Rate > 1 kHz se repiten muestras del acelerómetro.
- **DLPF** (*digital low-pass filter*, registro 26): filtra las muestras internas **antes** del divisor. Evita el aliasing de la decimación. No evita el del muestreo interno, que depende de la mecánica del sensor y del ADC sigma-delta, y no está documentado.

| `DLPF_CFG` | Acelerómetro: ancho de banda / retardo | Giróscopo: ancho de banda / retardo | Gyro Output Rate |
|---|---|---|---|
| 0 | 260 Hz / 0 ms | 256 Hz / 0,98 ms | 8 kHz |
| 1 | 184 Hz / 2,0 ms | 188 Hz / 1,9 ms | 1 kHz |
| 2 | 94 Hz / 3,0 ms | 98 Hz / 2,8 ms | 1 kHz |
| 3 | 44 Hz / 4,9 ms | 42 Hz / 4,8 ms | 1 kHz |
| 4 | 21 Hz / 8,5 ms | 20 Hz / 8,3 ms | 1 kHz |
| 5 | 10 Hz / 13,8 ms | 10 Hz / 13,4 ms | 1 kHz |
| 6 | 5 Hz / 19,0 ms | 5 Hz / 18,6 ms | 1 kHz |

- El **retardo** es de varios ms: el instante en que se lee una muestra no es el instante del fenómeno físico. Importa si se sincroniza con otra señal.
- Filtrar mucho por debajo de $f_s/2$ **correlaciona** muestras consecutivas → [[mediciones-incertezas#3.2 Cuándo deja de valer]].

---

## 3. Tiempo y jitter

### 3.1 Definiciones

Sea $t_k$ el instante medido del flanco $k$ y $T$ el período nominal.

| Nombre | Definición | Qué mira |
|---|---|---|
| TIE (*time interval error*) | $t_k - kT$, respecto de una grilla ideal | error de fase acumulado |
| *Period jitter* | dispersión de $P_k = t_{k+1} - t_k$ | variación de cada período |
| *Cycle-to-cycle jitter* | dispersión de $P_{k+1} - P_k$ | cambios bruscos entre períodos vecinos |
| Deriva de frecuencia | tendencia lenta de la media de $P_k$ | temperatura, envejecimiento |

### 3.2 Dos modelos de jitter

La misma σ de período puede venir de dos mecanismos que se promedian distinto:

| | Jitter de flancos | Jitter de períodos |
|---|---|---|
| Modelo | $t_k = kT + j_k$, con $j_k$ independientes | cada $P_k$ independiente |
| Ejemplo físico | reloj de cristal estable + ruido de disparo en cada flanco | oscilador cuya fase camina libremente |
| σ del período | $\sqrt2\,\sigma_j$ | $\sigma_P$ |
| Períodos consecutivos | correlación −½: uno largo, el siguiente corto | independientes |
| TIE | acotado | crece como $\sqrt k$ (*random walk*) |
| Media de N períodos | $(t_N - t_0)/N$: telescópica, σ baja como $1/N$ | σ baja como $1/\sqrt N$ |

Para distinguirlos: la autocorrelación de $P_k$ a distancia 1 (≈ −0,5 contra ≈ 0), o la varianza de Allan de los timestamps.

### 3.3 Timestamps cuantizados

Un instrumento que marca tiempos en una grilla de paso $\tau = 1/f_\text{instrumento}$ redondea cada instante.
- Si la fase de cada flanco respecto de la grilla es aleatoria e independiente, cada timestamp tiene error uniforme de ancho τ. La diferencia de dos (un período) es **triangular** en $[-\tau, \tau]$, con $\sigma = \tau/\sqrt6$ → [[probabilidad-distribuciones#Triangular]].
- Ese supuesto **falla** con una señal muy estable: la fase avanza de forma determinista y los errores de timestamps sucesivos no son independientes. El período medido toma solo dos valores vecinos de la grilla, en proporciones fijas. Si $T$ es un múltiplo exacto de τ, el error es cero en todos los períodos.
- Con jitter propio del orden de τ/2 o mayor, el jitter hace de dither y vale $\sigma_P^2 \approx 2\sigma_j^2 + \tau^2/6$.
- El error de medir una frecuencia contando $N$ períodos está acotado por la resolución de los dos extremos: $\lvert\delta f/f\rvert \lesssim \tau/(N T)$. Un registro largo lo hace despreciable; lo que queda es la exactitud del reloj del instrumento.

### 3.4 Estadística descriptiva y robusta

| Estimador | Robusto a outliers | Comentario |
|---|---|---|
| Media, σ | no | un solo valor grande domina σ |
| Mediana | sí | |
| MAD = mediana de $\lvert x_i - \text{mediana}\rvert$ | sí | para datos normales, $\sigma \approx 1{,}4826\cdot\text{MAD}$ |
| Percentiles (50, 99, 99,9) | sí | el 99,9 necesita muchas más de 1000 muestras |
| Máximo, conteo sobre un umbral | — | describen los eventos raros |

- Si σ y $1{,}4826\cdot\text{MAD}$ difieren mucho, hay colas pesadas u outliers. El contraste sirve como test automático.
- Ruido y eventos raros se informan **por separado**: σ o MAD para el ruido; conteo, tasa y máximo para los eventos.
- **Histograma:** el ancho del bin tiene que ser un múltiplo entero del paso del instrumento. Si no, aparecen bins vacíos o un patrón de peine que es artefacto de la cuantización.

---

## 4. Espectro y PSD

### 4.1 DFT

Con $N$ muestras a $f_s$, la DFT da componentes en frecuencias separadas por

$$
\Delta f = \frac{f_s}{N} = \frac{1}{\text{duración del registro}}
$$

desde 0 hasta $f_s/2$ (`numpy.fft.rfft` y `rfftfreq`). La **resolución en frecuencia** depende de cuánto tiempo se midió, no de $f_s$.

### 4.2 Fuga espectral y ventanas

La DFT supone que el bloque se repite periódicamente. Una frecuencia que no completa un número entero de ciclos en el bloque "se derrama" en los bins vecinos: es la **fuga espectral**. Multiplicar por una **ventana** (Hann, por ejemplo) que baja a cero en los bordes reduce la fuga a cambio de ensanchar el pico.

### 4.3 PSD

- **PSD** (*power spectral density*): cuánta varianza hay por unidad de frecuencia. Unidades: $x^2/\text{Hz}$.
- **ASD** (*amplitude spectral density*): su raíz, en $x/\sqrt{\text{Hz}}$. Es la "densidad de ruido" de las hojas de datos.
- **Welch:** partir el registro en segmentos, ventanear cada uno y promediar sus espectros. Da menos varianza en la estimación a cambio de menos resolución. `scipy.signal.welch` con `scaling="density"`.
- **Unilateral vs bilateral:** la unilateral (de 0 a $f_s/2$) vale el doble que la bilateral. Es la fuente clásica de errores de $\sqrt2$ o de 2.
- **Parseval (chequeo):** el área bajo la PSD unilateral entre 0 y $f_s/2$ tiene que dar la varianza de la señal.

### 4.4 De la densidad de ruido al valor rms

Para ruido blanco de ASD $n$ que pasa por un filtro de ancho de banda equivalente de ruido ENBW:

$$
\sigma = n\,\sqrt{\text{ENBW}}
$$

El ENBW es mayor que el ancho de banda a −3 dB: por un factor $\pi/2$ en un filtro RC de primer orden, y menos en órdenes mayores.

| Sensor (MPU6050, §6.1–6.2) | Densidad de ruido | Ancho de banda (−3 dB) | σ estimada |
|---|---|---|---|
| Acelerómetro | 400 µg/√Hz | 44 Hz | ≳ 2,7 mg |
| Giróscopo | 0,005 °/s/√Hz | 98 Hz | ≈ 0,05 °/s (la hoja da 0,05 °/s rms en esa configuración) |

---

## 5. Tipos de ruido

| Ruido | PSD $\propto$ | Allan $\sigma(\tau) \propto$ | Autocorrelación | Origen típico en una IMU |
|---|---|---|---|---|
| Cuantización | $f^{2}$ | $\tau^{-1}$ | — | ADC, formato numérico |
| Blanco | $f^{0}$ | $\tau^{-1/2}$ | delta en 0 | electrónica, ruido termomecánico |
| Flicker (1/f) | $f^{-1}$ | $\tau^{0}$ (plano) | decae muy lento | inestabilidad del sesgo |
| Random walk | $f^{-2}$ | $\tau^{+1/2}$ | no estacionaria | sesgo que "camina" |
| Rampa (deriva) | — | $\tau^{+1}$ | — | temperatura, calentamiento |
| Gauss–Markov | lorentziana | joroba | exponencial | procesos con una constante de tiempo |

**Por qué $\sigma/\sqrt N$ falla.** La fórmula supone autocorrelación delta (ruido blanco). Con flicker o random walk, la varianza muestral **depende de la duración del registro** y no converge: cuanto más se mide, más grande da. Hace falta un estimador que sí converja → sección 6.

---

## 6. Varianza de Allan

### 6.1 Definición

Se parte el registro en bloques de duración τ, se promedia cada bloque ($\bar y_k$) y se mide cuánto cambia el promedio de un bloque al siguiente:

$$
\sigma_A^2(\tau) = \tfrac12\,E\big[(\bar y_{k+1} - \bar y_k)^2\big]
$$

- Se calcula para muchos τ y se grafica $\sigma_A(\tau)$ en log-log.
- La versión **solapada** (*overlapping*) usa todos los bloques posibles y da más estadística con los mismos datos.
- Con `allantools`: `oadev(y, rate=fs, data_type="freq", taus=...)` para muestras de la magnitud (aceleración, velocidad angular, frecuencia fraccional); `data_type="phase"` para errores de tiempo.
- A diferencia de la varianza común, converge para flicker y random walk: por eso se usa.

### 6.2 Relación con la PSD

Con $S(f)$ la PSD unilateral:

$$
\sigma_A^2(\tau) = 2\int_0^\infty S(f)\,\frac{\sin^4(\pi f\tau)}{(\pi f\tau)^2}\,df
$$

Cada tipo de ruido da una pendiente característica (tabla de la sección 5). La curva se lee de izquierda a derecha: al principio domina el blanco (baja), después el flicker (piso plano), después el random walk o la deriva (sube).

### 6.3 Lectura de coeficientes (IEEE Std 952)

| Ruido | Pendiente | Cómo se lee | Nombre (giróscopo / acelerómetro) |
|---|---|---|---|
| Blanco | −½ | valor de la recta en τ = 1 s | ARW / VRW (*angle / velocity random walk*) |
| Flicker | 0 | $B = \sigma_\text{min}/0{,}664$ | inestabilidad de sesgo (*bias instability*) |
| Random walk | +½ | valor de la recta en τ = 3 s | RRW (*rate random walk*) |

**El factor √2.** Para ruido blanco con ASD **unilateral** $n$:

$$
\sigma_A^2(\tau) = \frac{n^2}{2\tau}
\qquad\Longrightarrow\qquad
\sigma_A(1\,\text{s}) = \frac{n}{\sqrt2}
$$

IEEE 952 escribe $\sigma_A^2 = N^2/\tau$ con la PSD bilateral, así que $N = n/\sqrt2$. Muchos fabricantes informan "ARW = densidad × 60" sin ese factor. Antes de comparar con una hoja de datos: generar ruido blanco con $n$ conocida y ver cuánto da $\sigma_A(1\ \text{s})$.

Unidades: (°/s)/√Hz × 60 = °/√h.

**Qué se puede comparar con la hoja de datos.** Si la hoja da solo densidad de ruido (como la MPU6050, a 10 Hz), solo el tramo de pendiente −½ tiene contra qué compararse. La inestabilidad de sesgo y el random walk son resultado propio.

### 6.4 Intervalo de confianza

Con $K$ bloques no solapados de duración τ, el error relativo de $\sigma_A$ es aproximadamente

$$
\frac{\delta\sigma_A}{\sigma_A} \approx \frac{1}{\sqrt{2(K-1)}}
$$

A τ grande quedan pocos bloques y las barras crecen. Regla práctica: τ máximo útil ≈ duración del registro / 10. Para ver el tramo que sube hacen falta registros de horas. Intervalos formales: χ² con los grados de libertad equivalentes que tabula Riley → [[probabilidad-distribuciones#χ² (chi cuadrado)]].

### 6.5 Origen

Nació para caracterizar relojes: la magnitud es la frecuencia fraccional $y = \delta f/f$. Un giróscopo quieto y un oscilador son el mismo problema: una salida que debería ser constante, con ruido de varios tipos. La misma herramienta aplicada a los timestamps de un sistema de adquisición caracteriza el reloj que marca $f_s$.

---

## 7. Filtros digitales

| Filtro | Ecuación | Ruido blanco: $\sigma^2$ de salida / entrada | En frecuencia |
|---|---|---|---|
| Promedio móvil de $M$ | $y_k = \frac1M\sum_{i=0}^{M-1} x_{k-i}$ | $1/M$ | tipo sinc: primer cero en $f_s/M$, −3 dB en ≈ $0{,}443\,f_s/M$ |
| IIR de primer orden | $y_k = \alpha x_k + (1-\alpha)\,y_{k-1}$ | $\alpha/(2-\alpha)$ | pasabajos de un polo; constante de tiempo $-T_s/\ln(1-\alpha) \approx T_s/\alpha$ |

- El promedio móvil es óptimo en el tiempo (baja el ruido conservando flancos) y malo en frecuencia (lóbulos laterales altos).
- El IIR con α = 1 no filtra: chequeo inmediato de cualquier implementación.
- **Decimar** = filtrar y después descartar muestras (sección 2.2).
- **Filtrar correlaciona las muestras.** Después de filtrar, $\sigma/\sqrt N$ sobre las muestras filtradas subestima la incerteza.

---

## 8. Resumen

| Idea | Ecuación |
|---|---|
| Error de cuantización (si el error barre el paso) | $\sigma_q = \Delta/\sqrt{12}$ |
| Resolución ≠ exactitud | LSB vs offset, ganancia y temperatura |
| Lo que está por encima de $f_s/2$ se pliega | $f_\text{alias} = \lvert f - f_s\operatorname{round}(f/f_s)\rvert$ |
| El antialiasing va antes de muestrear o decimar | — |
| Resolución en frecuencia = 1 / duración | $\Delta f = f_s/N$ |
| Densidad de ruido → rms | $\sigma = n\sqrt{\text{ENBW}}$ |
| Parseval como chequeo | $\int S(f)\,df = \sigma^2$ |
| Blanco en Allan (ASD unilateral $n$) | $\sigma_A(\tau) = n/\sqrt{2\tau}$ |
| Inestabilidad de sesgo | $B = \sigma_\text{min}/0{,}664$ |
| Error relativo de Allan con $K$ bloques | $1/\sqrt{2(K-1)}$ |

---

## 9. Preguntas de repaso (sin mirar)

1. ¿Por qué un ADC de 16 bits puede ser inexacto en varios cientos de LSB?
2. ¿Cuándo el error de cuantización deja de comportarse como una uniforme? ¿Qué lo "arregla"?
3. ¿Por qué un filtro digital no puede eliminar un alias? ¿Dónde tiene que estar el antialiasing?
4. En la MPU6050, ¿qué aliasing evita el DLPF y cuál no?
5. ¿Cómo distinguís jitter de flancos de jitter de períodos con los datos?
6. ¿Por qué guardar timestamps en float32 es un error?
7. Dada una densidad de ruido en µg/√Hz y un filtro, ¿cómo estimás la σ por muestra? ¿Por qué el ancho de banda a −3 dB no alcanza?
8. ¿Por qué la varianza común no sirve para ruido 1/f y la de Allan sí?
9. Dibujá una curva de Allan típica de una IMU y marcá qué se lee en cada tramo.
10. ¿De dónde sale el factor √2 entre una densidad de ruido y el ARW?

---

## Fuentes

- S. W. Smith, *The Scientist and Engineer's Guide to Digital Signal Processing* (gratis online): cap. 2 (estadística, histograma), cap. 3 (ADC, cuantización, teorema del muestreo), cap. 4 (punto fijo y flotante), cap. 8–9 (DFT, resolución, ventanas), cap. 15 (promedio móvil), cap. 19 (filtros recursivos).
- InvenSense PS-MPU-6000A, *MPU-6000 and MPU-6050 Product Specification*, rev. 3.2: §6.1 (giróscopo: ruido), §6.2 (acelerómetro: tolerancias, densidad de ruido), §6.6 (frecuencias de muestreo internas).
- InvenSense RM-MPU-6000A, *Register Map and Descriptions*: registro 25 (SMPLRT_DIV), 26 (CONFIG, tabla del DLPF).
- W. J. Riley, *Handbook of Frequency Stability Analysis*, NIST SP 1065 (gratis): varianza de Allan, versiones solapadas, grados de libertad e intervalos de confianza.
- IEEE Std 952, *Specification Format Guide and Test Procedure for Single-Axis Interferometric Fiber Optic Gyros*, anexo C: identificación de ruidos con la varianza de Allan y lectura de coeficientes.
- NIST/SEMATECH *e-Handbook of Statistical Methods*: §1.3.5.6 (medidas de escala: MAD).
- Nota de aplicación sobre jitter de un fabricante de relojes (Silicon Labs, Renesas o SiTime): definiciones de period jitter, cycle-to-cycle y TIE.
