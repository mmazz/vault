# Fundamentos eléctricos para buses y señales digitales

Resumen orientado a microcontroladores, GPIO, I²C, SPI, buses y electrónica digital.

---

# 1. Resistencia

La resistencia limita la corriente.

$$
\boxed{V = RI}
$$

$$
\boxed{I=\frac{V}{R}}
$$

Unidad:

$$
[R]=\Omega
$$

## Intuición

Una resistencia dificulta el movimiento de carga.

- $R\uparrow \Rightarrow I\downarrow$
- $V\uparrow \Rightarrow I\uparrow$

La resistencia disipa energía en forma de calor:

$$
\boxed{P=VI=I^2R=\frac{V^2}{R}}
$$

---

# 2. Capacitancia

Un capacitor almacena energía en un **campo eléctrico**.

Relación fundamental:

$$
\boxed{Q=CV}
$$

De:

$$
I=\frac{dQ}{dt}
$$

sale:

$$
\boxed{I=C\frac{dV}{dt}}
$$

Unidad:

$$
[C]=F
$$

En electrónica digital son comunes:

$$
pF,\quad nF,\quad \mu F
$$

## Intuición

Un capacitor se opone a cambios rápidos de **voltaje**.

$$
\boxed{\text{El voltaje de un capacitor no puede cambiar instantáneamente}}
$$

Para cambiar muy rápido $V$:

$$
\frac{dV}{dt}\uparrow
$$

se necesita mucha corriente:

$$
I=C\frac{dV}{dt}
$$

### Imagen mental

$$
\boxed{\text{Capacitor} \approx \text{inercia del voltaje}}
$$

No es una equivalencia física literal, pero es una buena intuición.

---

# 3. Energía almacenada en un capacitor

$$
\boxed{E_C=\frac{1}{2}CV^2}
$$

La energía está almacenada en el campo eléctrico.

---

# 4. Capacitancia parásita

No hace falta colocar físicamente un capacitor para tener capacitancia.

Cualquier par de conductores separados por un dieléctrico tiene cierta capacitancia.

Ejemplos:

- pistas de PCB;
- cables;
- patas de componentes;
- entradas de un MCU;
- entradas de sensores;
- breadboard;
- conectores.

Por eso una línea real puede verse aproximadamente como:

```text
signal -------------------------
          |      |      |
          C      C      C
          |      |      |
         GND    GND    GND
```

Podemos agruparlas:

$$
\boxed{
C_{\text{bus}}
=
C_1+C_2+C_3+\cdots
}
$$

## Intuición

Más dispositivos + más cable + más PCB:

$$
\boxed{C_{\text{bus}}\uparrow}
$$

y por lo tanto cuesta más cambiar rápidamente el voltaje de la línea.

---

# 5. Pull-up

Una resistencia pull-up conecta débilmente una señal con $V_{CC}$.

```text
VCC
 |
 Rpullup
 |
 +------ señal
```

Si nadie fuerza la línea:

$$
V_{\text{signal}}\approx V_{CC}
$$

por lo tanto tenemos un `1` lógico.

## Intuición

El pull-up:

> lleva la línea hacia `1` cuando nadie está controlándola activamente.

No es una fuente ideal de tensión: hay una resistencia en el medio.

---

# 6. ¿Por qué usar una resistencia?

Si conectáramos directamente:

```text
VCC
 |
 +------ signal
 |
 transistor
 |
GND
```

cuando el transistor conduce tendríamos prácticamente un cortocircuito.

Con una resistencia:

```text
VCC
 |
 R
 |
 +------ signal
 |
 transistor
 |
GND
```

la corriente queda limitada:

$$
\boxed{I=\frac{V_{CC}}{R}}
$$

Por ejemplo:

$$
V_{CC}=3.3V
$$

$$
R=4.7\,k\Omega
$$

entonces:

$$
I\approx0.70\,mA
$$

---

# 7. Push-pull

Una salida GPIO convencional suele poder conducir activamente la salida tanto hacia $V_{CC}$ como hacia GND.

Conceptualmente:

```text
       VCC
        |
    transistor
        |
GPIO ---+
        |
    transistor
        |
       GND
```

Puede producir activamente:

$$
0
$$

o:

$$
1
$$

Esto se conoce como:

$$
\boxed{\text{push-pull}}
$$

---

# 8. Open-drain

Una salida open-drain solamente tiene capacidad activa de llevar la línea hacia GND.

Conceptualmente:

```text
GPIO ----+
         |
     transistor
         |
        GND
```

Tiene dos estados.

### Transistor ON

$$
V_{\text{GPIO}}\approx0
$$

Produce un `0`.

### Transistor OFF

La salida queda desconectada eléctricamente.

Eso se llama:

$$
\boxed{\text{alta impedancia / Hi-Z}}
$$

No produce activamente un `1`.

---

# 9. Open-drain + pull-up

Al combinar ambos:

```text
        VCC
         |
      Rpullup
         |
         +--------- SDA
         |
      transistor
         |
        GND
```

tenemos:

### Transistor ON

$$
SDA=0
$$

### Transistor OFF

El pull-up lleva la línea a:

$$
SDA\approx V_{CC}
$$

Por lo tanto:

$$
\boxed{
\text{open-drain}
=
\text{forzar 0 o soltar la línea}
}
$$

---

# 10. ¿Por qué I²C usa open-drain?

Porque varios dispositivos comparten SDA y SCL.

```text
                VCC
                 |
               Rpullup
                 |
SDA -------------+----------------
        |             |           |
       MCU          sensor       sensor
        |             |           |
       SW            SW          SW
        |             |           |
       GND           GND         GND
```

Todos pueden tirar la línea hacia cero.

Ninguno intenta imponer activamente un `1`.

Por eso no existe el conflicto:

```text
dispositivo A -> fuerza HIGH
dispositivo B -> fuerza LOW
```

que con push-pull podría provocar una corriente muy grande.

### Regla conceptual de I²C

$$
\boxed{
0 = \text{alguien está tirando la línea a GND}
}
$$

$$
\boxed{
1 = \text{nadie está tirando la línea a GND}
}
$$

---

# 11. Pull-up + capacitancia = circuito RC

Una línea I²C real se parece aproximadamente a:

```text
            VCC
             |
           Rpullup
             |
             +------ SDA
             |
            Cbus
             |
            GND
```

Esto es un circuito RC.

Su constante de tiempo es:

$$
\boxed{\tau=RC}
$$

---

# 12. Carga del capacitor

Si inicialmente:

$$
V(0)=0
$$

y soltamos la línea, el pull-up carga la capacitancia:

$$
\boxed{
V(t)=V_{CC}
\left(
1-e^{-t/RC}
\right)
}
$$

Valores útiles:

$$
t=\tau \Rightarrow V\approx0.63V_{CC}
$$

$$
t=3\tau \Rightarrow V\approx0.95V_{CC}
$$

$$
t=5\tau \Rightarrow V\approx0.99V_{CC}
$$

---

# 13. Rise time

Una señal digital real no hace:

```text
      ┌────────
      │
──────┘
```

sino aproximadamente:

```text
          _______
       .´
     .´
   .´
_.´
```

El tiempo que tarda en subir se llama:

$$
\boxed{t_r=\text{rise time}}
$$

Para un circuito RC:

$$
\boxed{t_r\propto RC}
$$

Por lo tanto:

$$
R\uparrow \Rightarrow t_r\uparrow
$$

$$
C\uparrow \Rightarrow t_r\uparrow
$$

---

# 14. Capacitancia del bus y velocidad

Si aumentamos:

$$
C_{\text{bus}}
$$

la línea tarda más en alcanzar un `1` lógico.

Por eso:

$$
\boxed{
C_{\text{bus}}\uparrow
\Rightarrow
t_{\text{rise}}\uparrow
\Rightarrow
f_{\max}\downarrow
}
$$

La capacitancia limita principalmente la **velocidad máxima** del bus.

---

# 15. Elegir el pull-up es un compromiso

Resistencia grande:

$$
R\uparrow
$$

produce:

$$
I\downarrow
$$

pero:

$$
RC\uparrow
$$

por lo que la subida es más lenta.

Resistencia pequeña:

$$
R\downarrow
$$

produce subida más rápida, pero cuando la línea está LOW:

$$
I=\frac{V_{CC}}{R}
$$

aumenta.

Por lo tanto:

$$
\boxed{
R_{\text{pullup}}\text{ grande}
\Rightarrow
\text{menos corriente, subida lenta}
}
$$

$$
\boxed{
R_{\text{pullup}}\text{ pequeña}
\Rightarrow
\text{más corriente, subida rápida}
}
$$

---

# 16. Inductancia

Un conductor por el que circula corriente genera un campo magnético.

Una inductancia almacena energía en ese campo magnético.

La relación fundamental es:

$$
\boxed{
V=L\frac{dI}{dt}
}
$$

Unidad:

$$
[L]=H
$$

---

# 17. Intuición de la inductancia

Una inductancia se opone a cambios rápidos de **corriente**.

$$
\boxed{
\text{La corriente de un inductor no puede cambiar instantáneamente}
}
$$

Para cambiar muy rápido la corriente:

$$
\frac{dI}{dt}\uparrow
$$

hace falta una tensión grande:

$$
V=L\frac{dI}{dt}
$$

### Imagen mental

$$
\boxed{
\text{Inductor}\approx\text{inercia de la corriente}
}
$$

---

# 18. Energía almacenada en una inductancia

$$
\boxed{
E_L=\frac12LI^2
}
$$

La energía está almacenada en el campo magnético.

---

# 19. Capacitor vs inductor

| | Capacitor | Inductor |
|---|---|---|
| Campo | eléctrico | magnético |
| Almacena según | voltaje | corriente |
| Ecuación | $I=C\,dV/dt$ | $V=L\,dI/dt$ |
| No permite cambiar instantáneamente | $V$ | $I$ |
| Energía | $\frac12CV^2$ | $\frac12LI^2$ |
| Intuición | inercia del voltaje | inercia de la corriente |

---

# 20. Inductancia parásita

Todo conductor real tiene inductancia.

Por ejemplo un cable:

```text
MCU ---------------- sensor
```

puede modelarse mejor como:

```text
MCU ---- L ---- R ---- sensor
```

Hay inductancia en:

- cables;
- pistas;
- vias;
- conectores;
- patas de componentes;
- planos de alimentación.

Por eso existe:

$$
\boxed{L_{\text{parásita}}}
$$

aunque nunca hayamos colocado una bobina.

---

# 21. ¿Por qué importa en electrónica digital?

Porque:

$$
\boxed{V=L\frac{dI}{dt}}
$$

Un MCU puede cambiar corrientes extremadamente rápido.

Aunque $L$ sea pequeña, si:

$$
\frac{dI}{dt}
$$

es grande, aparece una tensión apreciable.

Ejemplo:

$$
L=20\,nH
$$

$$
\Delta I=20\,mA
$$

$$
\Delta t=10\,ns
$$

Entonces:

$$
V\approx L\frac{\Delta I}{\Delta t}
$$

$$
V
=
20\times10^{-9}
\frac{20\times10^{-3}}
{10\times10^{-9}}
$$

$$
\boxed{V\approx40\,mV}
$$

---

# 22. Lo importante no es sólo la frecuencia

Una señal puede ser de solamente:

$$
1\,kHz
$$

pero el GPIO puede cambiar de LOW a HIGH en:

$$
5\,ns
$$

El circuito tiene que responder al flanco de 5 ns.

Por eso:

$$
\boxed{
\text{frecuencia de señal}
\neq
\text{velocidad del flanco}
}
$$

Y en integridad de señal muchas veces importa más:

$$
\boxed{t_r,\;t_f}
$$

que la frecuencia fundamental.

---

# 23. Capacitancia + inductancia

Todo circuito real tiene:

$$
R,\qquad L,\qquad C
$$

aunque no hayamos colocado explícitamente esos componentes.

Una interconexión real se parece más a:

```text
            L       R
signal ----^^^^----/\/\-------
                         |
                         C
                         |
                        GND
```

que a un cable matemáticamente ideal.

---

# 24. Circuito LC y resonancia

Una inductancia y una capacitancia pueden intercambiar energía.

La frecuencia natural ideal es:

$$
\boxed{
f_0=
\frac{1}
{2\pi\sqrt{LC}}
}
$$

La energía oscila entre:

$$
E_C=\frac12CV^2
$$

y:

$$
E_L=\frac12LI^2
$$

---

# 25. Ringing

Un flanco rápido puede excitar las $L$ y $C$ parásitas.

En vez de:

```text
      ┌────────────
      │
──────┘
```

podemos observar:

```text
       /\_/\/\______
      /
_____/
```

Esto se llama:

$$
\boxed{\text{ringing}}
$$

Es una oscilación transitoria causada por la energía almacenada en las inductancias y capacitancias del circuito.

La resistencia normalmente amortigua la oscilación.

---

# 26. Overshoot y undershoot

Debido a las inductancias y capacitancias parásitas, una señal puede superar momentáneamente sus valores nominales.

### Overshoot

$$
V>V_{CC}
$$

### Undershoot

$$
V<0
$$

Ejemplo:

```text
          /\
3.3 V ---/  \______
       /
______/
```

Eso puede ser relevante porque los pines de los MCUs tienen límites máximos y mínimos de tensión.

---

# 27. RC, RL y RLC

## RC

Constante de tiempo:

$$
\boxed{\tau=RC}
$$

Asociado principalmente con cambios de voltaje.

## RL

Constante de tiempo:

$$
\boxed{\tau=\frac{L}{R}}
$$

Asociado principalmente con cambios de corriente.

## RLC

Combina:

- almacenamiento eléctrico $C$;
- almacenamiento magnético $L$;
- disipación $R$.

Puede producir oscilaciones amortiguadas.

---

# 28. Otra intuición útil: R, C y L

### Resistencia

$$
\boxed{R:\text{ disipa energía}}
$$

### Capacitor

$$
\boxed{C:\text{ almacena energía eléctrica}}
$$

### Inductor

$$
\boxed{L:\text{ almacena energía magnética}}
$$

El capacitor y el inductor pueden devolver la energía almacenada.

La resistencia no: la transforma fundamentalmente en calor.

---

# 29. Corriente y carga

Relación fundamental:

$$
\boxed{
I=\frac{dQ}{dt}
}
$$

La corriente es simplemente:

> cantidad de carga que atraviesa una sección por unidad de tiempo.

Integrando:

$$
\boxed{
Q=\int I\,dt
}
$$

Esto hace especialmente intuitiva la ecuación del capacitor:

$$
Q=CV
$$

porque:

$$
I=C\frac{dV}{dt}
$$

---

# 30. Alta impedancia

Una entrada o salida en alta impedancia:

$$
\boxed{Z\rightarrow\text{muy grande}}
$$

idealmente no entrega ni absorbe corriente significativa.

No significa necesariamente `0`.

No significa necesariamente `1`.

Significa aproximadamente:

> estoy eléctricamente desconectado de esta línea.

Por eso un pin flotante puede tomar valores impredecibles.

---

# 31. Floating input

Una entrada sin conexión definida:

```text
GPIO input ---- nada
```

está:

$$
\boxed{\text{floating}}
$$

Puede captar ruido electromagnético y terminar siendo interpretada alternativamente como `0` o `1`.

Se soluciona normalmente con:

- pull-up;
- pull-down;
- una fuente que controle activamente la señal.

---

# 32. Pull-down

Es el equivalente inverso del pull-up:

```text
signal
 |
 R
 |
GND
```

Si nadie controla la señal:

$$
V\approx0
$$

Por lo tanto establece un estado lógico por defecto LOW.

---

# 33. Rise time y fall time

Se definen:

$$
t_r = \text{tiempo de subida}
$$

$$
t_f = \text{tiempo de bajada}
$$

En I²C pueden ser bastante diferentes.

Con open-drain:

### HIGH → LOW

El transistor descarga activamente:

$$
\boxed{\text{bajada rápida}}
$$

### LOW → HIGH

El pull-up carga $C_{\text{bus}}$:

$$
\boxed{\text{subida RC más lenta}}
$$

Esta asimetría es característica de I²C.

---

# 34. Idea importante sobre cables

A baja velocidad podemos imaginar:

```text
entrada ---------------- salida
```

como si todo el cable tuviera instantáneamente el mismo voltaje.

Pero físicamente las señales electromagnéticas se propagan a velocidad finita.

Cuando el tiempo de propagación del cable deja de ser despreciable frente al rise time:

$$
t_{\text{prop}}
\sim
t_r
$$

el cable empieza a comportarse como una:

$$
\boxed{\text{línea de transmisión}}
$$

y ya no alcanza con pensar solamente en $R$, $L$ y $C$ concentrados en un punto.

---

# 35. Impedancia

En continua usamos principalmente resistencia:

$$
V=RI
$$

Pero cuando las señales cambian con el tiempo, capacitores e inductores también afectan la relación entre tensión y corriente.

Se usa entonces el concepto:

$$
\boxed{
Z=\text{impedancia}
}
$$

Para una resistencia:

$$
\boxed{Z_R=R}
$$

Para un capacitor:

$$
\boxed{
Z_C=\frac{1}{j\omega C}
}
$$

Para un inductor:

$$
\boxed{
Z_L=j\omega L
}
$$

---

# 36. Intuición frecuencial

Para un capacitor:

$$
|Z_C|=\frac{1}{\omega C}
$$

entonces:

$$
f\uparrow
\Rightarrow
|Z_C|\downarrow
$$

### Intuición

El capacitor ofrece un camino cada vez más fácil para componentes de alta frecuencia.

Para un inductor:

$$
|Z_L|=\omega L
$$

entonces:

$$
f\uparrow
\Rightarrow
|Z_L|\uparrow
$$

### Intuición

El inductor dificulta cada vez más las variaciones rápidas de corriente.

---

# 37. Por qué aparecen capacitores de desacople

Cerca de un MCU normalmente encontrás capacitores como:

$$
100\,nF
$$

entre:

$$
V_{CC}
$$

y:

$$
GND
$$

Su objetivo es proporcionar localmente corriente durante cambios rápidos.

```text
VCC ----+------ MCU
        |
       === C
        |
       GND
```

Si el MCU necesita un pequeño pulso rápido de corriente, el capacitor cercano puede suministrarlo.

### Intuición

La fuente de alimentación puede estar físicamente lejos.

El capacitor es una pequeña reserva de energía ubicada al lado del chip.

$$
\boxed{
\text{decoupling capacitor}
\approx
\text{reservorio local para transitorios rápidos}
}
$$

---

# 38. ¿Por qué tiene que estar cerca del MCU?

Porque las pistas entre la fuente y el MCU tienen inductancia:

$$
V=L\frac{dI}{dt}
$$

Una corriente que cambia rápidamente no puede llegar instantáneamente desde una fuente distante sin generar perturbaciones de tensión.

Por eso:

$$
\boxed{
\text{capacitor de desacople cerca del pin}
}
$$

reduce el área del camino de corriente y la inductancia efectiva.

---

# 39. Idea física general

Una señal digital ideal:

```text
0 → 1
```

parece puramente lógica.

Pero físicamente significa mover carga:

$$
Q=CV
$$

en un intervalo de tiempo:

$$
I=\frac{dQ}{dt}
$$

por conductores que tienen inductancia:

$$
V=L\frac{dI}{dt}
$$

y resistencia:

$$
V=RI
$$

Por lo tanto, detrás de cada transición digital están:

$$
\boxed{
R,\quad L,\quad C
}
$$

---

# 40. Resumen mental

## Resistencia

$$
\boxed{V=RI}
$$

**Limita corriente y disipa energía.**

## Capacitor

$$
\boxed{I=C\frac{dV}{dt}}
$$

**Se opone a cambios rápidos de voltaje.**

$$
\boxed{E_C=\frac12CV^2}
$$

## Inductor

$$
\boxed{V=L\frac{dI}{dt}}
$$

**Se opone a cambios rápidos de corriente.**

$$
\boxed{E_L=\frac12LI^2}
$$

## RC

$$
\boxed{\tau=RC}
$$

**Controla qué tan rápido puede cambiar un voltaje mediante una resistencia.**

## Open-drain

$$
\boxed{\text{LOW o Hi-Z}}
$$

No genera HIGH activamente.

## Pull-up

$$
\boxed{\text{lleva la línea a HIGH cuando nadie la tira a LOW}}
$$

## I²C

$$
\boxed{
\text{open-drain + pull-up + }C_{\text{bus}}
}
$$

produce una subida aproximadamente RC.

## Capacitancia parásita

$$
\boxed{
C_{\text{bus}}\uparrow
\Rightarrow
t_r\uparrow
\Rightarrow
f_{\max}\downarrow
}
$$

## Inductancia parásita

$$
\boxed{
V=L\frac{dI}{dt}
}
$$

Cambios rápidos de corriente pueden producir picos de tensión.

## LC

$$
\boxed{
f_0=\frac{1}{2\pi\sqrt{LC}}
}
$$

Puede producir ringing.

---

# 41. Las cuatro intuiciones que conviene recordar

$$
\boxed{
Q=CV
}
$$

**Para cambiar el voltaje de una capacitancia hay que mover carga.**

$$
\boxed{
I=\frac{dQ}{dt}
}
$$

**Mover esa carga rápidamente requiere corriente.**

$$
\boxed{
V=L\frac{dI}{dt}
}
$$

**Cambiar una corriente rápidamente requiere tensión.**

$$
\boxed{
\text{digital rápido}
\Rightarrow
\text{problemas analógicos}
}
$$

Una señal es digital desde el punto de vista de la información.

Desde el punto de vista físico sigue siendo una señal electromagnética gobernada por circuitos analógicos.
