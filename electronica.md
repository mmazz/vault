# Fundamentos eléctricos para buses digitales

Orientado a lo que usa el proyecto: GPIO, I²C (IMU, LCD), SPI (MAX31865) y lo que muestran el analizador lógico y el osciloscopio. Supone física de circuitos: lo que ya sabés de electromagnetismo va comprimido en la sección 1.

---

## 0. La idea que ordena todo

Un 0 → 1 es información, pero físicamente es **mover carga** ($Q = CV$) en un tiempo finito ($I = dQ/dt$) a través de conductores con **resistencia** ($V = RI$) e **inductancia** ($V = L\,dI/dt$).

De ahí sale todo lo que sigue:

| Efecto | Causa | Sección |
|---|---|---|
| La subida lenta de I²C | pull-up cargando la capacidad del bus | 4 |
| Oscilaciones y picos en flancos rápidos | L y C parásitas | 5 |
| Capacitores de desacople | L de la alimentación | 6 |
| Lo que el instrumento muestra (y lo que agrega) | muestreo, umbral, carga de la punta | 7 |

> Una señal es digital desde el punto de vista de la información; físicamente es un circuito analógico. Digital rápido ⇒ problemas analógicos.

---

## 1. R, C y L

| | Resistencia | Capacitor | Inductor |
|---|---|---|---|
| Relación | $V = RI$ | $I = C\,dV/dt$ (de $Q = CV$ e $I = dQ/dt$) | $V = L\,dI/dt$ |
| Qué no puede saltar | — | la tensión | la corriente |
| Energía | disipa: $P = I^2R = V^2/R$ | almacena $\tfrac12 CV^2$ (campo eléctrico) | almacena $\tfrac12 LI^2$ (campo magnético) |
| Impedancia | $Z_R = R$ | $Z_C = 1/(j\omega C)$ | $Z_L = j\omega L$ |
| Con la frecuencia | constante | $\lvert Z_C\rvert$ baja: camino fácil para lo rápido | $\lvert Z_L\rvert$ sube: frena lo rápido |
| Imagen mental | fricción | inercia de la tensión | inercia de la corriente |
| Escalas típicas acá | Ω–kΩ | pF (pines, cables), nF (desacople) | nH (cables, pistas) |

Consecuencias que se usan después:

- Cambiar rápido la tensión de una capacidad exige mucha corriente; cambiar rápido una corriente a través de una inductancia exige mucha tensión.
- Constantes de tiempo: $\tau = RC$ y $\tau = L/R$.
- RLC serie: frecuencia natural $f_0 = 1/(2\pi\sqrt{LC})$; oscila (subamortiguado) si $R < 2\sqrt{L/C}$. C y L intercambian energía; R la disipa.

### 1.1 Kirchhoff, serie y paralelo

- **Corrientes (KCL):** en un nodo, $\sum I = 0$. Es conservación de la carga.
- **Tensiones (KVL):** en un lazo, $\sum V = 0$. Vale si el potencial está bien definido, o sea si no hay flujo magnético variable a través del lazo. Cuando lo hay, se modela como una **L** en el lazo: es la inductancia parásita de la sección 2.

| | Serie | Paralelo |
|---|---|---|
| Qué es igual en todos | la corriente | la tensión |
| $R$ | $R_1 + R_2 + \cdots$ | $1/R_{eq} = 1/R_1 + 1/R_2 + \cdots$; dos: $R_1R_2/(R_1+R_2)$ |
| $C$ | $1/C_{eq} = 1/C_1 + 1/C_2 + \cdots$ | $C_1 + C_2 + \cdots$ |
| $L$ | $L_1 + L_2 + \cdots$ (sin acople) | $1/L_{eq} = 1/L_1 + 1/L_2 + \cdots$ |

Las capacidades en paralelo se suman: por eso cada dispositivo, cable e instrumento que se cuelga de un bus suma a $C_b$ (secciones 2 y 4).

### 1.2 Divisor resistivo y carga

```text
Vin
 |
 R1
 |
 +------ Vout
 |
 R2
 |
GND
```

Sin carga:

$$
V_{out} = V_{in}\,\frac{R_2}{R_1 + R_2}
$$

**Con carga.** Visto desde la salida, el divisor es una fuente de Thévenin:

$$
V_{th} = V_{in}\,\frac{R_2}{R_1+R_2},
\qquad
R_{th} = R_1 \parallel R_2 = \frac{R_1R_2}{R_1+R_2}
$$

Con una carga $R_L$ en la salida:

$$
V_{out} = V_{th}\,\frac{R_L}{R_{th} + R_L}
\qquad\Longrightarrow\qquad
\frac{\delta V_{out}}{V_{out}} \approx -\frac{R_{th}}{R_L} \quad (R_L \gg R_{th})
$$

| $R_1 = R_2$ | $R_{th}$ | Corriente en reposo (3,3 V) | Error con $R_L = 1$ MΩ |
|---|---|---|---|
| 10 kΩ | 5 kΩ | 165 µA | −0,5 % |
| 100 kΩ | 50 kΩ | 16,5 µA | −4,8 % |

- **Compromiso:** R grandes consumen poco, pero el divisor queda más sensible a la carga y más ruidoso ($R_{th}$ grande). R chicas, al revés.
- **ADC de muestreo:** en cada conversión conecta un capacitor interno a la entrada y lo carga desde $R_{th}$. Si $R_{th}$ es grande, no llega a cargarse en el tiempo de muestreo y la lectura queda baja. Un capacitor externo en la salida hace de reserva local (y además filtra, sección 1.5).
- **Incerteza del cociente:** con $k = R_2/(R_1+R_2)$ y tolerancias relativas independientes $u_1$, $u_2$,

$$
\frac{u_k}{k} = \frac{R_1}{R_1+R_2}\,\sqrt{u_1^2 + u_2^2}
$$

  Con $R_1 = R_2$ al 1 %: ~0,7 % en el cociente. Un error común a las dos resistencias (misma deriva térmica) se cancela en el cociente → [[mediciones-incertezas#4. Propagación]].

### 1.3 Transitorio y régimen permanente

Después de un escalón, C y L se comportan distinto en el primer instante y al final:

| Elemento | Justo después del escalón ($t = 0^+$) | Régimen permanente DC ($t \to \infty$) |
|---|---|---|
| Capacitor | cortocircuito: su tensión no puede saltar | circuito abierto: $I = C\,dV/dt = 0$ |
| Inductor | circuito abierto: su corriente no puede saltar | cortocircuito: $V = L\,dI/dt = 0$ |

"DC" describe el estado final, no la conexión: al encender o conmutar siempre hay un transitorio.

### 1.4 Impedancia, reactancia y fase

Para señales senoidales de frecuencia angular $\omega = 2\pi f$:

$$
Z = \frac{V}{I} = R + jX,
\qquad
\lvert Z\rvert = \sqrt{R^2 + X^2},
\qquad
\varphi = \operatorname{atan}\frac{X}{R}
$$

| Elemento | $Z$ | Reactancia $X$ | Fase de la corriente respecto de la tensión |
|---|---|---|---|
| R | $R$ | 0 | en fase |
| C | $1/(j\omega C)$ | $-1/(\omega C)$ | adelanta 90° |
| L | $j\omega L$ | $\omega L$ | atrasa 90° |

- La parte **real** disipa energía. La **reactiva** la almacena y la devuelve en cada ciclo: en promedio no consume. La potencia media es $\bar P = V_{rms}\,I_{rms}\cos\varphi$.
- La impedancia cambia la amplitud **y** la fase. Un filtro retrasa la señal además de atenuarla.

### 1.5 RC como filtro pasabajos

```text
Vin --- R ---+--- Vout
             |
             C
             |
            GND
```

$$
H(f) = \frac{V_{out}}{V_{in}} = \frac{1}{1 + j\,f/f_c},
\qquad
f_c = \frac{1}{2\pi RC} = \frac{1}{2\pi\tau}
$$

| Frecuencia | Módulo | Fase |
|---|---|---|
| $f \ll f_c$ | ≈ 1 | ≈ 0° |
| $f = f_c$ | $1/\sqrt2$ (−3 dB) | −45° |
| $f \gg f_c$ | ≈ $f_c/f$: cae 20 dB por década | → −90° |

Tiempo y frecuencia son dos caras del mismo τ:
- **Tiempo de subida:** $t_r(10\text{–}90\,\%) = RC\ln 9 \approx 2{,}2\,\tau$. En función de $f_c$: $t_r \approx 0{,}35/f_c$. De ahí sale el $0{,}35/t_r$ de la sección 5.1.
- **Ancho de banda de ruido:** $\text{ENBW} = (\pi/2)\,f_c$ → [[senales-ruido#4.4 De la densidad de ruido al valor rms]].
- Intercambiando R y C se obtiene un **pasaaltos**, con el mismo $f_c$: bloquea la continua y deja pasar las variaciones (acople de alterna).

Usos: filtro antialiasing antes de un ADC → [[senales-ruido#2.2 El antialiasing va antes]], antirrebote de pulsadores, reset al encender, filtrado de ruido en una entrada.

### 1.6 Escalón en un RL

$$
I(t) = \frac{V}{R}\left(1 - e^{-tR/L}\right),
\qquad
\tau = \frac{L}{R}
$$

Al **cortar** la corriente de una carga inductiva (relé, motor, solenoide), $dI/dt$ se hace enorme y $V = L\,dI/dt$ produce un pico de tensión que puede destruir el transistor que la maneja. Por eso se pone un **diodo de rueda libre** en paralelo con la carga: le da un camino a la corriente mientras la energía $\tfrac12 LI^2$ se disipa.

### 1.7 Resonancia y analogía mecánica

En resonancia, $\lvert X_L\rvert = \lvert X_C\rvert$, es decir $\omega_0 L = 1/(\omega_0 C)$, y la energía pasa entre C y L sin que la fuente tenga que aportar la parte reactiva.

| RLC serie | Masa–resorte–amortiguador |
|---|---|
| carga $q$ | posición $x$ |
| corriente $I$ | velocidad $v$ |
| $L$ | masa $m$ |
| $1/C$ | constante del resorte $k$ |
| $R$ | amortiguamiento $c$ |
| $f_0 = 1/(2\pi\sqrt{LC})$ | $f_0 = \frac{1}{2\pi}\sqrt{k/m}$ |
| $Q = \frac1R\sqrt{L/C}$ | $Q = \sqrt{km}/c$ |

Subamortiguado ($R < 2\sqrt{L/C}$) es lo mismo que $Q > 1/2$: la respuesta a un escalón oscila. Es la misma ecuación que la de un acelerómetro MEMS.

---

## 2. Parásitos: R, L y C que nadie puso

Todo conductor real tiene R, L y C, aunque no haya componentes con esos nombres.

**Capacidad parásita.** Aparece entre cualquier par de conductores separados por un dieléctrico:
- entradas de los chips (UM10204 admite hasta 10 pF por pin de I/O);
- cables, pistas, contactos contiguos de la protoboard, conectores;
- la punta o el canal del instrumento que conectás para medir.

En un bus, todas quedan en paralelo contra masa y se suman: $C_b = \sum C_i$. Más dispositivos, más cable, más instrumento ⇒ $C_b$ mayor.

**Inductancia parásita.** Aparece en cables, pistas, vías, patas y conectores. Lo que importa no es el conductor aislado sino el **lazo** que recorre la corriente: ida por la señal, vuelta por masa. Cuanto más área encierra el lazo, más L. Por eso importa que la masa vaya cerca de la señal (y por eso el cable de masa de una punta es un problema, sección 7).

**Modelo de una interconexión real:**

```text
            L        R
señal ---^^^^^^---/\/\/\---+--- entrada
                           |
                           C
                           |
                          GND
```

Este modelo "concentrado" vale mientras el cable sea corto frente al flanco (sección 5.4).

---

## 3. Estados de un pin

| Modo | ¿Fuerza 0? | ¿Fuerza 1? | Uso típico |
|---|---|---|---|
| Push-pull | sí | sí | GPIO de salida, SPI, TX de UART |
| Open-drain | sí | no: queda en Hi-Z | I²C, líneas compartidas |
| Entrada (Hi-Z) | no | no | lectura |

```text
push-pull:                    open-drain:
      VCC
       |
  transistor                  pin ----+
       |                              |
pin ---+                         transistor
       |                              |
  transistor                         GND
       |
      GND
```

- **Alta impedancia (Hi-Z).** El pin no entrega ni absorbe corriente apreciable. No es 0 ni 1: es "estoy desconectado de esta línea".
- **Entrada flotante.** Una entrada CMOS es básicamente la capacidad de una compuerta. Sin nada que fije su tensión, se carga por fugas y acoplamiento, y lee 0 o 1 al azar. Si queda en una tensión intermedia, los dos transistores de la etapa de entrada conducen a medias y consume de más.
- **Pull-up / pull-down.** Una resistencia a VCC o a GND que fija el estado por defecto. Es "débil": cualquier salida activa le gana.
- **Por qué resistencia y no un cable a VCC.** Cuando un transistor tira la línea a 0, la corriente queda limitada a $I = (V_{CC} - V_{OL})/R \approx V_{CC}/R$. Con 3,3 V y 4,7 kΩ, ≈ 0,7 mA. Con un cable, sería un cortocircuito.
- **Push-pull en una línea compartida.** Si un dispositivo fuerza 1 y otro fuerza 0, la corriente solo la limitan las resistencias internas de los transistores (contención). Por eso los buses compartidos usan open-drain.

---

## 4. I²C eléctrico

### 4.1 Open-drain + pull-up = "wired-AND"

```text
               3,3 V
                 |
             R_pull-up
                 |
SDA -------------+-----------------+-----------------
                 |                 |                 |
               ESP32              IMU            otro disp.
                 |                 |                 |
             transistor        transistor        transistor
                 |                 |                 |
                GND               GND               GND
```

- **0** = al menos un dispositivo tira la línea a GND.
- **1** = nadie la tira; la pull-up la lleva a VCC.
- Nadie genera un 1 activamente. SCL funciona igual.

Dos razones para que sea open-drain:

1. **No hay contención:** en el peor caso la línea queda en 0; nunca hay un cortocircuito entre dispositivos.
2. **Cualquiera puede tomar una línea que maneja otro:** el esclavo contesta sobre SDA (ACK y datos), un esclavo lento estira SCL (*clock stretching*) y, con varios masters, se arbitra: el que suelta la línea (quiere un 1) y lee 0 sabe que perdió.

### 4.2 Dónde está la pull-up

Puede estar en el ESP32 (internas, ≈ 45 kΩ según su hoja de datos, se activan por software), en el módulo del sensor o como resistencia externa. **Todas las que estén conectadas quedan en paralelo:**

$$\frac{1}{R_{eq}} = \sum_i \frac{1}{R_i}$$

- Módulo con 4,7 kΩ + internas del ESP32 activadas → ≈ 4,3 kΩ.
- Dos módulos con 4,7 kΩ cada uno (IMU + adaptador del LCD en la fase 5) → 2,35 kΩ.

Qué pull-ups hay en tu bus se averigua (capa 0 de 1.1); no se supone.

### 4.3 Niveles lógicos

UM10204 define los umbrales en proporción a la alimentación (tabla de características de SDA y SCL):

| | Valor |
|---|---|
| $V_{IL}$ (máximo que se lee como 0) | $0{,}3\,V_{DD}$ |
| $V_{IH}$ (mínimo que se lee como 1) | $0{,}7\,V_{DD}$ |
| $V_{OL}$ (0 que tiene que garantizar quien tira a masa) | ≤ 0,4 V absorbiendo 3 mA |

Entre $V_{IL}$ y $V_{IH}$ el nivel no está definido. Esto importa en 4.4: el tiempo de subida se mide justamente entre esos dos umbrales.

### 4.4 El flanco de subida es la carga de un RC

Cuando todos sueltan la línea, la pull-up carga $C_b$:

```text
   VDD
    |
    R  (pull-up equivalente)
    |
    +------ SDA
    |
   C_b
    |
   GND
```

$$V(t) = V_{DD}\left(1 - e^{-t/RC}\right) \qquad\Longrightarrow\qquad t(x) = -RC\,\ln(1-x)$$

donde $x$ es la fracción de $V_{DD}$ alcanzada. Valores útiles: 63 % en $\tau$, 95 % en $3\tau$, 99 % en $5\tau$.

**Tiempo de subida.** Depende de entre qué fracciones se mida:

| Definición | Resultado | Dónde se usa |
|---|---|---|
| 10 % → 90 % | $RC\ln 9 \approx 2{,}2\,RC$ | definición genérica (osciloscopios, hojas de datos) |
| 30 % → 70 % | $RC\ln(0{,}7/0{,}3) \approx 0{,}8473\,RC$ | I²C: son sus umbrales $V_{IL}$ y $V_{IH}$ |

**Límites de UM10204:** $t_r \le 1000$ ns en modo estándar (100 kHz), $t_r \le 300$ ns en modo rápido (400 kHz); $C_b \le 400$ pF en ambos.

Con $C_b = 100$ pF:

| Pull-up | $t_r$ (30–70 %) | 100 kHz | 400 kHz |
|---|---|---|---|
| Interna ESP32, ≈ 45 kΩ | ≈ 3,8 µs | no | no |
| 10 kΩ | ≈ 850 ns | sí | no |
| 4,7 kΩ | ≈ 400 ns | sí | no |
| 2,2 kΩ | ≈ 190 ns | sí | sí |

### 4.5 Elegir la pull-up: dos límites

- **Máximo, por velocidad:** $R_{max} = t_{r,max} / (0{,}8473\,C_b)$. Con 100 pF: ≈ 11,8 kΩ a 100 kHz, ≈ 3,5 kΩ a 400 kHz. Depende de $C_b$.
- **Mínimo, por corriente:** quien tira a 0 tiene que absorber la corriente de la pull-up y seguir garantizando $V_{OL}$: $R_{min} = (V_{DD} - V_{OL,max}) / I_{OL} = (3{,}3 - 0{,}4)\,\text{V} / 3\,\text{mA} \approx 970\ \Omega$. Depende de $V_{DD}$.

| | R grande | R chica |
|---|---|---|
| Subida | lenta | rápida |
| Corriente con la línea en 0 | baja | alta |
| Inmunidad al ruido | peor (línea "débil") | mejor |

Procedimiento completo: UM10204, §7.1 "Pull-up resistor sizing"; TI SLVA689.

### 4.6 Asimetría subida/bajada

- **Bajada:** un transistor descarga $C_b$ activamente → rápida (del orden de ns).
- **Subida:** la pull-up carga $C_b$ → exponencial, lenta.

En el osciloscopio, I²C se ve como flancos de bajada abruptos y subidas redondeadas. La que limita la velocidad del bus es la subida:

$$C_b \uparrow \;\Rightarrow\; t_r \uparrow \;\Rightarrow\; f_{max} \downarrow$$

---

## 5. Flancos rápidos

### 5.1 Importa el flanco, no la frecuencia

Una señal de 1 kHz cuyo GPIO conmuta en 5 ns tiene contenido espectral hasta aproximadamente

$$f \approx \frac{0{,}35}{t_r} \quad (t_r \text{ de } 10\text{–}90\,\%) \;\;\Rightarrow\;\; 5\ \text{ns} \to 70\ \text{MHz}$$

El circuito responde al flanco de 5 ns, no a la fundamental de 1 kHz. En integridad de señal importan $t_r$ y $t_f$.

### 5.2 Inductancia parásita: $V = L\,dI/dt$

Aunque L sea chica, una $dI/dt$ grande produce una tensión apreciable. Ejemplo: $L = 20$ nH, $\Delta I = 20$ mA en $\Delta t = 10$ ns:

$$V \approx L\,\frac{\Delta I}{\Delta t} = 20\ \text{nH}\cdot\frac{20\ \text{mA}}{10\ \text{ns}} = 40\ \text{mV}$$

**Ground bounce:** si varias salidas conmutan juntas y devuelven su corriente por la misma pata de masa, las $dI/dt$ se suman y la masa interna del chip se mueve respecto de la de la placa.

### 5.3 Ringing, overshoot y undershoot

Un flanco rápido excita las L y C parásitas, que intercambian energía; la R del camino la disipa.

```text
ideal:              real:
     ┌──────             /\  /\
     │                  /  \/  \/‾‾‾‾‾   ← overshoot + ringing
─────┘              ___/
```

- Oscila si el camino está subamortiguado: $R < 2\sqrt{L/C}$. Ejemplo ilustrativo: $L = 100$ nH y $C = 100$ pF dan $f_0 \approx 50$ MHz y $2\sqrt{L/C} \approx 63\ \Omega$. Una salida de pocas decenas de ohms con ese cable oscila.
- **Overshoot** ($V > V_{DD}$) y **undershoot** ($V < 0$): los pines tienen límites (sección *Absolute Maximum Ratings* de cada hoja de datos). Fuera de ellos conducen los diodos de protección internos.
- En I²C, el flanco que puede oscilar es la **bajada** (rápida), no la subida RC. UM10204 fija un $t_f$ mínimo en modo rápido para limitar este efecto.
- En SPI (MAX31865, fase 4) las salidas son push-pull: los dos flancos son rápidos.

### 5.4 Cuándo un cable deja de ser "un nodo"

La señal se propaga a velocidad finita: $v \approx c/\sqrt{\varepsilon_{ef}}$, del orden de 15–20 cm/ns en cables y pistas. El modelo concentrado (sección 2) vale mientras el tiempo de propagación sea chico frente al flanco. Regla práctica: $t_{prop} \lesssim t_r/6$ (según la fuente, entre 1/4 y 1/10). Longitud crítica:

$$\ell_{crit} \approx \frac{v\,t_r}{6}$$

| Flanco | $t_r$ | $\ell_{crit}$ |
|---|---|---|
| Subida I²C | ~300 ns | decenas de metros: irrelevante |
| Bajada I²C, flancos de SPI | ~10 ns | del orden de 30 cm |

Por encima de $\ell_{crit}$ el cable es una **línea de transmisión**: importan su impedancia característica, las reflexiones y la terminación (la terminación de 120 Ω de CAN en la fase 7 es exactamente esto). Consecuencia práctica: un cable dupont largo ya está en zona gris para los flancos rápidos.

---

## 6. Alimentación: capacitores de desacople

Cada conmutación del chip pide un pulso de corriente de pocos ns. La fuente está lejos y entre medio hay inductancia de cables y pistas: $V = L\,dI/dt$ hace caer la tensión en el pin justo cuando el chip la necesita.

```text
fuente ---- L (cables, pistas) ----+------ VDD del chip
                                   |
                                 === 100 nF, pegado al pin
                                   |
                                  GND
```

- El capacitor de desacople (típicamente 100 nF por pin de alimentación) es una **reserva local** para transitorios rápidos.
- **Tiene que estar cerca** porque lo que importa es el área del lazo capacitor–pin–masa: cuanto menor, menos L entre la reserva y el chip.
- Se complementa con uno de mayor valor (µF) para transitorios más lentos. El capacitor real también tiene inductancia propia (ESL), por eso se combinan valores.
- Tus módulos (ESP32, IMU) ya los traen. Sirve para leer sus esquemáticos y para reconocer ruido de alimentación en una medición.

---

## 7. Qué ve cada instrumento (y qué le hace al circuito)

**Analizador lógico.** Compara la tensión con un umbral y muestrea: solo ve 0 o 1 en instantes discretos.
- A 24 MHz, una muestra cada 41,7 ns. Un pulso más corto puede perderse; un flanco se ubica con esa granularidad.
- No ve $t_r$, ringing ni niveles de tensión.
- Con una subida lenta, el instante en que "ve" el flanco depende de su umbral. En un flanco RC con $\tau = 4{,}5$ µs (pull-up interna, 100 pF), pasar del 30 % al 50 % de $V_{DD}$ son ≈ 1,5 µs. El umbral de tu analizador es un dato a averiguar, no a suponer.

**Osciloscopio.** Ve la forma analógica: $t_r$, $t_f$, ringing, niveles.
- Su propio tiempo de subida es ≈ $0{,}35/\text{BW}$. Para medir un $t_r$ hace falta que el del instrumento sea varias veces menor.

**Los dos cargan el circuito.**
- La punta o el canal suman capacidad al bus (una punta ×10 típica, del orden de 10 pF; ver su especificación). Medís el bus con el instrumento conectado.
- Un cable de masa largo en la punta forma un lazo con L: el ringing que aparece puede ser de la medición, no del circuito.

---

## 8. Level shifter

Cuando dos partes del bus tienen distinta alimentación (por ejemplo, ESP32 a 3,3 V y un módulo a 5 V), la pull-up del lado de 5 V pondría 5 V en un pin de 3,3 V. Si eso supera sus *Absolute Maximum Ratings* (verificarlo en la hoja de datos del ESP32), hay que separar los dominios.

El level shifter bidireccional típico para I²C usa un MOSFET por línea y una pull-up de cada lado, a su propia tensión (NXP AN10441). Funciona justamente porque el bus es open-drain: nadie empuja un 1, cada lado lo pone con su propia pull-up. Es relevante en la fase 5 (LCD).

---

## 9. Resumen

| Idea | Ecuación |
|---|---|
| Para cambiar la tensión de una capacidad hay que mover carga | $Q = CV$ |
| Moverla rápido requiere corriente | $I = dQ/dt = C\,dV/dt$ |
| Cambiar rápido una corriente requiere tensión | $V = L\,dI/dt$ |
| La subida de I²C es un RC y limita la velocidad | $t_r = 0{,}8473\,RC$; $C_b\uparrow \Rightarrow f_{max}\downarrow$ |
| La pull-up está acotada por los dos lados | $\dfrac{V_{DD}-V_{OL}}{I_{OL}} \le R \le \dfrac{t_{r,max}}{0{,}8473\,C_b}$ |
| L y C parásitas oscilan con flancos rápidos | $f_0 = 1/(2\pi\sqrt{LC})$, subamortiguado si $R < 2\sqrt{L/C}$ |
| El flanco, no la frecuencia, fija el ancho de banda | $f \approx 0{,}35/t_r$ |
| Un divisor cargado cae en $R_{th}/R_L$ | $R_{th} = R_1 \parallel R_2$ |
| RC: el mismo τ en tiempo y en frecuencia | $f_c = 1/(2\pi RC)$, $t_r \approx 2{,}2\,RC = 0{,}35/f_c$ |
| Cortar una corriente inductiva genera un pico | $V = L\,dI/dt$ → diodo de rueda libre |

---

## 10. Preguntas de repaso (sin mirar)

1. ¿Por qué la subida de I²C es exponencial y la bajada no?
2. ¿Por qué en I²C el $t_r$ se mide entre 30 % y 70 %, y de dónde sale el 0,8473?
3. ¿Qué dos límites acotan la pull-up y de qué parámetro depende cada uno?
4. Si agregás el LCD al bus de la IMU, ¿qué pasa con $R_{eq}$, con $C_b$ y con $t_r$?
5. ¿Una señal de 1 kHz puede tener problemas de integridad de señal? ¿Por qué?
6. ¿Qué puede y qué no puede mostrarte el analizador lógico de un flanco de SDA?
7. ¿Por qué el capacitor de desacople tiene que estar cerca del pin, si la tensión es la misma en todo el cable?
8. Un divisor de 100 kΩ + 100 kΩ entra a un ADC. ¿Qué dos efectos hacen que la lectura quede baja, y cómo se corrigen?
9. ¿Cómo se comportan C y L justo después de un escalón y en régimen permanente?
10. Un RC tiene $f_c = 1$ kHz. ¿Cuánto vale su tiempo de subida? ¿Y su ENBW?
11. ¿Por qué hace falta un diodo en paralelo con la bobina de un relé?

---

## Fuentes

- NXP UM10204, *I2C-bus specification and user manual*: tabla de características de SDA y SCL ($V_{IL}$, $V_{IH}$, $V_{OL}$, $t_r$, $t_f$, $C_b$, $C_i$); §7.1 "Pull-up resistor sizing".
- TI SLVA689, *I2C Bus Pullup Resistor Calculation*.
- NXP AN10441, *Level shifting techniques in I2C-bus design*.
- Espressif, hoja de datos del ESP32: pull-ups internas, *Absolute Maximum Ratings*.
- H. Johnson y M. Graham, *High-Speed Digital Design: A Handbook of Black Magic*: ancho de banda de un flanco, longitud crítica, ringing, desacople.
- P. Horowitz y W. Hill, *The Art of Electronics*, 3.ª ed., cap. 1: Kirchhoff, divisor y Thévenin, RC en tiempo y en frecuencia, impedancia, diodo de rueda libre.
