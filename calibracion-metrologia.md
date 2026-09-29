# Calibración y metrología de laboratorio

Cómo se calibra un sensor, cómo se sostiene esa calibración con una cadena de referencias, cómo se decide si algo cumple una especificación y cómo se valida un instrumento antes de creerle. Las herramientas de incerteza están en [[mediciones-incertezas]].

---

## 0. La idea que ordena todo

**Calibrar** es comparar la indicación de un instrumento con una referencia de valor conocido, en condiciones controladas, y expresar la relación entre ambas **con su incerteza**. No es "dejarlo bien": eso es ajustar.

| Pregunta | Herramienta | Sección |
|---|---|---|
| ¿Qué es calibrar y qué no? | calibración, ajuste, verificación | 1 |
| ¿Contra qué se compara y por qué creerle? | trazabilidad | 2 |
| ¿Qué parámetros tiene un sensor y cómo se estiman? | modelo de calibración | 3 |
| ¿Cómo se calibra una cadena de temperatura? | RTD, simulación eléctrica | 4 |
| ¿Cómo se escribe el resultado? | presupuesto de laboratorio | 5 |
| ¿Cumple o no cumple? | reglas de decisión | 6 |
| ¿El instrumento que uso es suficientemente bueno? | validación, TUR | 7 |

> Una calibración sin incertidumbre no es una calibración: es una comparación.

---

## 1. Calibración, ajuste y verificación

| | Qué es | Toca el instrumento | VIM |
|---|---|---|---|
| **Calibración** | establecer la relación entre los valores de un patrón y las indicaciones, con incertezas | no | 2.39 |
| **Ajuste** | modificar el instrumento (o sus coeficientes) para que indique mejor | sí | 3.11 |
| **Verificación** | comprobar que cumple requisitos especificados | no | 2.44 |

- En lenguaje coloquial "calibrar" suele querer decir ajustar. En una entrevista de calibración se nota la diferencia.
- Lo habitual es: calibrar → calcular coeficientes → ajustar (cargar los coeficientes) → **volver a calibrar** para verificar que el ajuste funcionó.

---

## 2. Trazabilidad

**Trazabilidad metrológica** (VIM 2.41): relación del resultado con una referencia a través de una **cadena ininterrumpida y documentada** de calibraciones, cada una con su incerteza.

```text
SI
 └─ instituto nacional de metrología (en Argentina: INTI)
     └─ laboratorio acreditado bajo ISO/IEC 17025 (en Argentina acredita el OAA)
         └─ patrón de trabajo
             └─ instrumento
```

- La incerteza **crece** a lo largo de la cadena: cada eslabón suma la suya.
- **ISO/IEC 17025:** norma de competencia de laboratorios de ensayo y calibración. Un laboratorio **acreditado** fue auditado contra ella para un alcance definido.
- **CMC** (*calibration and measurement capability*): la menor incerteza que un laboratorio acreditado puede ofrecer para una magnitud y un rango. Está en su alcance de acreditación.

**Certificado de calibración** (ISO/IEC 17025, §7.8): identificación del ítem, método, condiciones ambientales, resultados **con incertidumbre expandida y factor k**, declaración de trazabilidad y fecha. No lleva "vencimiento": el intervalo de recalibración lo decide el usuario.

**Sin certificados no hay trazabilidad** en sentido estricto. Lo correcto es decir "referencia independiente, con incertidumbre Tipo B tomada de la especificación del fabricante".

---

## 3. Modelo de calibración de un sensor

### 3.1 Parámetros

Para un sensor de 3 ejes (acelerómetro, giróscopo, magnetómetro):

$$
\mathbf{m} = K\,\mathbf{x} + \mathbf{b} + \varepsilon
$$

| Símbolo | Qué es | Parámetros |
|---|---|---|
| $\mathbf b$ | **sesgo** u offset: lo que indica con entrada cero | 3 |
| diagonal de $K$ | **factores de escala**: cuánto indica por unidad de entrada | 3 |
| fuera de la diagonal de $K$ | **desalineación** y no ortogonalidad: cuánto ve cada eje de los otros | 6 |
| $\varepsilon$ | ruido → [[senales-ruido#5. Tipos de ruido]] | — |

Todos dependen de la **temperatura**: una calibración vale a la temperatura a la que se hizo, salvo que se modele esa dependencia.

### 3.2 Acelerómetro: la gravedad como patrón

Con el sensor quieto, cada eje se apunta a $+g$ y a $-g$ (seis posiciones).

- Estimador simple, sin desalineación, para el eje $i$:

$$
b_i = \frac{m_i^{\uparrow} + m_i^{\downarrow}}{2},
\qquad
S_i = \frac{m_i^{\uparrow} - m_i^{\downarrow}}{2g}
$$

- Completo: en cada posición se registran los tres ejes. Son 18 ecuaciones para 12 incógnitas, que se resuelven por cuadrados mínimos con matriz de covarianza → [[probabilidad-distribuciones#Ajuste por cuadrados mínimos]]. Las ecuaciones sobrantes permiten chequear el modelo con los residuos.

### 3.3 La g local

- **Gravedad normal** (fórmula de Somigliana, elipsoide GRS80/WGS84) a la latitud de Buenos Aires (~34,6° S): ≈ 9,797 m/s². Corrección de aire libre: $-3{,}086\times10^{-6}$ m/s² por metro de altura.
- Las anomalías locales son de decenas de mGal: ~$10^{-5}$ relativo. Hay valores medidos en el *Gravity Information System* del PTB y en la red gravimétrica del IGN.

| Fuente de error | Tamaño | ¿Importa? |
|---|---|---|
| Usar $g_n = 9{,}80665$ en lugar de la local | ~0,1 % ≈ 1 mg por g | sí: es comparable al paso de varios LSB |
| Incerteza de la g local bien calculada | ~$10^{-5}$ | no |
| Superficie inclinada 1°, en el eje alineado | $g(1-\cos\theta) \approx 0{,}15$ mg (segundo orden) | poco |
| Superficie inclinada 1°, en los ejes cruzados | $g\sin\theta \approx 17$ mg (primer orden) | mucho, para la desalineación |
| Cambio de temperatura entre posiciones | depende del coeficiente térmico del cero | registrar la temperatura |

El nivelado afecta poco al sesgo y a la escala, y mucho a la desalineación estimada.

---

## 4. Temperatura con RTD

### 4.1 Callendar–Van Dusen

Una **PT100** es un RTD (*resistance temperature detector*) de platino con $R_0 = 100\ \Omega$ a 0 °C. No es una termocupla. Para $t \ge 0$ °C (IEC 60751):

$$
R(t) = R_0\,(1 + A\,t + B\,t^2),
\qquad
A = 3{,}9083\times10^{-3}\ ^\circ\text{C}^{-1},
\quad
B = -5{,}775\times10^{-7}\ ^\circ\text{C}^{-2}
$$

Por debajo de 0 °C se agrega el término $C\,(t-100)\,t^3$, con $C = -4{,}183\times10^{-12}\ ^\circ\text{C}^{-4}$. El "α = 0,00385" que se cita es la pendiente media entre 0 y 100 °C, $(R_{100} - R_0)/(100\,R_0)$, no la ecuación.

| $t$ (°C) | $R$ (Ω) | $dR/dt$ (Ω/°C) | Error si se usa α lineal (°C) |
|---|---|---|---|
| 0 | 100,000 | 0,391 | 0 |
| 25 | 109,735 | 0,388 | −0,28 |
| 50 | 119,397 | 0,385 | −0,38 |
| 100 | 138,506 | 0,379 | −0,02 |

La aproximación lineal cuesta hasta ~0,4 °C a temperatura ambiente, del orden de una clase B entera.

### 4.2 Clases de tolerancia (IEC 60751)

| Clase | Tolerancia (°C) | A 25 °C |
|---|---|---|
| AA | $\pm(0{,}10 + 0{,}0017\lvert t\rvert)$ | ±0,14 |
| A | $\pm(0{,}15 + 0{,}002\lvert t\rvert)$ | ±0,20 |
| B | $\pm(0{,}30 + 0{,}005\lvert t\rvert)$ | ±0,43 |

Sin calibración propia, la clase entra como Tipo B uniforme: $u = \text{tolerancia}/\sqrt3$.

### 4.3 Conexión: 2, 3 y 4 hilos

| Hilos | Resistencia de los cables | Comentario |
|---|---|---|
| 2 | se suma a la del sensor | 1 Ω de cable (ida y vuelta) ≈ 2,6 °C de error |
| 3 | se compensa **suponiendo cables iguales** | queda el desbalance entre cables |
| 4 | no interviene: corriente por un par, tensión por el otro | la de laboratorio |

### 4.4 Medición radiométrica (MAX31865)

La resistencia de referencia $R_\text{ref}$ y el RTD están en serie con la misma corriente, y el ADC mide la **razón** de las dos tensiones:

$$
R_\text{RTD} = \frac{\text{código}}{2^{15}}\,R_\text{ref}
$$

- La tensión de referencia se cancela. **$R_\text{ref}$ no:** su tolerancia y su deriva térmica pasan directo al resultado. Una $R_\text{ref}$ de 0,1 % equivale a ~0,28 °C a 25 °C.
- Resolución: 15 bits, 0,03125 °C nominal.
- Exactitud total especificada del integrado: 0,5 °C (0,05 % del fondo de escala). Revisar en la hoja qué incluye.

### 4.5 Autocalentamiento

La corriente de medición disipa $P = I^2 R$ en el sensor, que queda más caliente que lo que mide:

$$
\Delta T = \frac{P}{\delta}
$$

donde $\delta$ es la **constante de disipación** en mW/°C, que depende del elemento y del montaje (va de unos pocos mW/°C para elementos chicos de película delgada a decenas para elementos grandes, TI SBAA310). Con unos mA en 100 Ω, la potencia es del orden del mW, y el error puede ser de décimas de grado.
- Se reduce bajando la corriente o encendiendo la excitación solo durante la conversión.
- Se mide: misma temperatura con dos corrientes (o con excitación continua y por pulsos); la diferencia es el autocalentamiento.

### 4.6 Equilibrio térmico

La constante de tiempo es $\tau = mc/(hA)$. Después de un escalón, el error cae como $e^{-t/\tau}$: a 5τ queda 0,7 %. Si dos sensores con distinta τ miden lo mismo mientras la temperatura cambia, el gráfico de uno contra otro muestra **histéresis** que no es de ninguno de los dos: es el desfasaje.

### 4.7 Calibración por simulación eléctrica (EURAMET cg-11)

Se calibran por separado el **indicador** y el **sensor**.
1. **Indicador** (electrónica + $R_\text{ref}$ + firmware + cables): se reemplaza el RTD por resistencias de valor conocido que "simulan" temperaturas (por ejemplo, 109,735 Ω simula 25 °C). La $R_\text{ref}$ queda absorbida en esta calibración.
2. **Sensor:** entra con su clase de tolerancia (Tipo B) o con una calibración propia en un baño o bloque.

Términos típicos del presupuesto: certificado de las resistencias de referencia, resolución, repetibilidad, cables, autocalentamiento, clase o calibración del sensor, equilibrio térmico, modelo (CVD vs lineal).

### 4.8 Otros sensores de temperatura

| Sensor | Principio | Lo típico | Cuidado con |
|---|---|---|---|
| Termocupla | efecto Seebeck: tensión por un gradiente de temperatura en el conductor | rango enorme; decenas de µV/°C | mide **diferencia** con la junta fría → compensación de junta fría |
| NTC | la resistencia cae exponencialmente con T | muy sensible, barato | no lineal: Steinhart–Hart, $1/T = a + b\ln R + c(\ln R)^3$ |
| Sensor en el chip | propiedad de un semiconductor | ya integrado | mide la temperatura **del chip**, no la del ambiente |

---

## 5. Presupuestos de laboratorio

EA-4/02 es la guía que usan los laboratorios de calibración europeos para aplicar el GUM. Aporta:
- la **forma estándar de la tabla**: magnitud, estimación, incerteza estándar, distribución, coeficiente de sensibilidad, contribución → [[mediciones-incertezas#6.2 Tabla]];
- los divisores Tipo B de uso diario → [[mediciones-incertezas#6.3 Del dato al Tipo B]];
- cuándo $k = 2$ es válido y qué hacer con pocos grados de libertad;
- **ejemplos resueltos** en sus suplementos.

Para aprenderla, lo que sirve es rehacer un ejemplo cercano (temperatura, resistencia o tensión) hasta reproducir cada número de la tabla.

---

## 6. Conformidad: ¿cumple o no?

### 6.1 El problema

"¿Está dentro de la especificación?" con un resultado que tiene incertidumbre. Cerca del límite, la respuesta es una **probabilidad**. Para un resultado $y$ con incerteza $u$ (normal) y límites $[T_L, T_U]$:

$$
P(\text{conforme}) = \Phi\!\left(\frac{T_U - y}{u}\right) - \Phi\!\left(\frac{T_L - y}{u}\right)
$$

### 6.2 Reglas de decisión (ILAC-G8)

| Regla | Pasa si… | Riesgo de falsa aceptación | Costo |
|---|---|---|---|
| Aceptación simple ($w = 0$) | el valor medido está dentro del límite | hasta ~50 % justo en el límite | — |
| Banda de guarda $w = U$ | está dentro del límite achicado en $U$ | ≤ ~2,5 % con $k = 2$ | más falsos rechazos |

- **Falsa aceptación:** declarar conforme algo que no lo es. **Falso rechazo:** lo contrario. La banda de guarda cambia uno por el otro.
- La regla se acuerda **antes** de medir y se declara en el informe.

### 6.3 Peor caso vs RSS

Para combinar tolerancias:
- **Peor caso:** sumar los límites. Garantiza, pero es pesimista.
- **RSS** (raíz de la suma de cuadrados): supone independencia y da un total ~$\sqrt n$ veces más chico para $n$ términos iguales.

Es la misma elección que suma lineal vs cuadratura → [[mediciones-incertezas#5. Suma lineal o en cuadratura]].

### 6.4 Capacidad de proceso

$$
C_p = \frac{\text{USL} - \text{LSL}}{6\sigma},
\qquad
C_{pk} = \min\left(\frac{\text{USL} - \mu}{3\sigma},\ \frac{\mu - \text{LSL}}{3\sigma}\right)
$$

- $C_p$ mira solo el ancho; $C_{pk}$ también el centrado.
- $C_{pk} = 1$: ~1350 ppm fuera de especificación de un lado (normal). $C_{pk} = 1{,}33$ es un objetivo típico.
- Supone un proceso **estable** y aproximadamente **normal**. Verificar las dos cosas antes de informarlo.

### 6.5 Tasas de pasa/no pasa y comparaciones

- **Tasa de fallas:** binomial. Con 0 fallas en $n$ ensayos, la cota al 95 % es ≈ $3/n$ → [[probabilidad-distribuciones#Resumen de momentos]].
- **Comparar dos grupos:** t de Student para medias, F para varianzas, Kolmogorov–Smirnov para distribuciones completas.
- **Significativo no es relevante:** con muchos datos, cualquier diferencia minúscula da significativa. Lo que decide es si importa frente a la tolerancia.

---

## 7. Validación de instrumentos

### 7.1 TUR

**TUR** (*test uncertainty ratio*): tolerancia de lo que se quiere verificar dividida por la incertidumbre expandida ($k = 2$) de la medición.

- La regla histórica pide TUR ≥ 4:1.
- ANSI/NCSL Z540.3 la reemplaza por un criterio de probabilidad: falsa aceptación ≤ 2 %.
- Con TUR < 1, el instrumento no puede decidir nada: su propia incertidumbre es mayor que lo que se quiere ver.

### 7.2 La referencia tiene que ser independiente

Validar un instrumento con una señal generada por el mismo sistema que después va a medir es **circular**: un error común se cancela y no se ve. La referencia tiene que tener su propia especificación o calibración.

### 7.3 Qué validar en cada instrumento

| Instrumento | Qué validar | Contra qué | Nota |
|---|---|---|---|
| Analizador lógico | exactitud de la base de tiempo (ppm) | contador de frecuencia u osciloscopio con especificación conocida; señal de 1 PPS de un GPS | un clon sin especificación exige un supuesto Tipo B justificado, que suele dominar el presupuesto de tiempo |
| Cadena de temperatura | el indicador completo | resistencias de valor conocido (sección 4.7) | |
| Medidor de corriente con shunt | offset y ganancia | multímetro | ver abajo |
| Multímetro | — | es la referencia: su "±(% de lectura + dígitos)" es Tipo B uniforme | no es trazable sin certificado |

**Corriente con shunt (INA226, hoja de datos, características eléctricas):** LSB de 2,5 µV en el shunt y 1,25 mV en el bus; offset del shunt ±10 µV máx.; error de ganancia ±0,1 % máx.; tiempo de conversión de 140 µs a 8,244 ms, con promediado programable.
- $I = V_\text{shunt}/R_\text{shunt}$: la tolerancia y la deriva térmica del shunt entran como Tipo B.
- A corriente baja domina el **offset** (10 µV en 0,1 Ω son 100 µA). A corriente alta, la **ganancia** y el shunt.

---

## 8. Resumen

| Idea | Ecuación o regla |
|---|---|
| Calibrar no es ajustar | calibración (VIM 2.39) ≠ ajuste (3.11) ≠ verificación (2.44) |
| Trazabilidad = cadena documentada con incertezas | SI → INTI → laboratorio acreditado → patrón → instrumento |
| Modelo de un sensor de 3 ejes | $\mathbf m = K\mathbf x + \mathbf b + \varepsilon$ (12 parámetros) |
| Sesgo y escala con ±g | $b = (m^\uparrow + m^\downarrow)/2$, $S = (m^\uparrow - m^\downarrow)/2g$ |
| g local, no estándar | la diferencia es ~0,1 % |
| PT100 | $R = R_0(1 + At + Bt^2)$; lineal cuesta ~0,4 °C |
| Radiométrico: la referencia de tensión se cancela, $R_\text{ref}$ no | $R = (\text{código}/2^{15})\,R_\text{ref}$ |
| Autocalentamiento | $\Delta T = I^2R/\delta$ |
| Riesgo en el límite | aceptación simple ~50 %; banda $w = U$ ≤ ~2,5 % |
| Capacidad | $C_{pk} = \min(\text{USL}-\mu, \mu-\text{LSL})/3\sigma$ |
| Instrumento suficiente | TUR ≥ 4 o falsa aceptación ≤ 2 % |

---

## 9. Preguntas de repaso (sin mirar)

1. ¿Cuál es la diferencia entre calibrar, ajustar y verificar? ¿En qué orden se hacen?
2. ¿Por qué la incerteza crece a lo largo de una cadena de trazabilidad?
3. ¿Qué parámetros tiene el modelo de calibración de un sensor de 3 ejes y cuántas posiciones hacen falta como mínimo?
4. ¿Por qué el nivelado de la superficie afecta poco al factor de escala y mucho a la desalineación?
5. En una medición radiométrica, ¿qué error se cancela y cuál no?
6. ¿Cómo medirías el autocalentamiento de un RTD?
7. ¿Qué ventaja tiene calibrar el indicador por simulación eléctrica, separado del sensor?
8. Un resultado cae justo en el límite de especificación. ¿Cuál es el riesgo con aceptación simple? ¿Qué cambia con una banda de guarda?
9. ¿Por qué una validación hecha con una señal del mismo sistema puede no detectar nada?

---

## Fuentes

- JCGM 200:2012 (VIM): 2.39 (calibración), 2.41 (trazabilidad), 2.44 (verificación), 3.11 (ajuste).
- ISO/IEC 17025:2017, *General requirements for the competence of testing and calibration laboratories*: §7.8 (informe de resultados). Solo conceptos.
- EA-4/02 M:2022, *Evaluation of the Uncertainty of Measurement in Calibration*, y sus ejemplos resueltos.
- EURAMET cg-11, *Guidelines on the Calibration of Temperature Indicators and Simulators by Electrical Simulation and Measurement*.
- IEC 60751, *Industrial platinum resistance thermometers and platinum temperature sensors*: ecuación de Callendar–Van Dusen y clases de tolerancia.
- Analog Devices (Maxim), hoja de datos del MAX31865: medición radiométrica, resolución, exactitud, configuración de 2/3/4 hilos.
- Texas Instruments SBAA310: selección de corriente de excitación y autocalentamiento de RTD.
- Texas Instruments SBOS547, hoja de datos del INA226: tabla de características eléctricas.
- ILAC-G8:09/2019, *Guidelines on Decision Rules and Statements of Conformity*.
- JCGM 106:2012, *The role of measurement uncertainty in conformity assessment*.
- ANSI/NCSL Z540.3, *Requirements for the Calibration of Measuring and Test Equipment*: TUR y probabilidad de falsa aceptación.
- NIST/SEMATECH *e-Handbook of Statistical Methods*: §6.1.6 (capacidad de proceso).
