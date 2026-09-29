# Mediciones e incertezas

Qué es una incerteza, cómo se reduce, cómo se combina y cómo se informa. Usa el vocabulario de un laboratorio de calibración (VIM, GUM), que es el que se usa en la industria. Las herramientas de probabilidad están en [[probabilidad-distribuciones]].

---

## 0. La idea que ordena todo

El valor verdadero de una magnitud no se conoce. Lo que se conoce es una **distribución** de valores compatibles con lo que medí y con lo que sé del instrumento. La **incerteza estándar** $u$ es la desviación estándar de esa distribución.

| Pregunta | Herramienta | Sección |
|---|---|---|
| ¿Qué es una incerteza y qué no? | vocabulario; error vs incerteza | 1, 2 |
| ¿Cuánto mejora promediar? | $\sigma/\sqrt N$ y sus límites | 3 |
| ¿Cómo se combinan varias fuentes? | propagación; suma lineal o en cuadratura | 4, 5 |
| ¿Cómo se informa un resultado? | presupuesto GUM | 6 |

> La incerteza no es un rango garantizado: es una σ. Con distribución normal, $\pm u$ contiene el valor en ~68 % de los casos.

---

## 1. Vocabulario (VIM)

| Término | VIM | Qué es | No confundir |
|---|---|---|---|
| Mensurando | 2.3 | la magnitud que se quiere medir, definida con suficiente detalle | "la temperatura" no es un mensurando; "la temperatura del chip a los 10 min de encendido" sí |
| Error de medida | 2.16 | valor medido − valor de referencia | si la referencia es el valor verdadero, el error no se conoce |
| Error sistemático | 2.17 | componente que se repite igual o varía de forma predecible | se puede corregir; la corrección tiene incerteza |
| Sesgo | 2.18 | estimación de un error sistemático | un offset es un sesgo |
| Error aleatorio | 2.19 | componente que cambia de forma impredecible entre repeticiones | promediar lo reduce solo si es independiente (sección 3) |
| Exactitud | 2.13 | cercanía al valor verdadero | es cualitativa: no lleva número |
| Veracidad | 2.14 | cercanía de la media de muchas repeticiones a la referencia | es lo contrario del sesgo |
| Precisión | 2.15 | cercanía entre repeticiones | un instrumento puede ser preciso y estar sesgado |
| Incertidumbre | 2.26 | parámetro no negativo que caracteriza la dispersión de los valores atribuibles al mensurando | no es el error |
| Evaluación Tipo A | 2.28 | por análisis estadístico de valores medidos | habla del **método**, no del tipo de efecto |
| Evaluación Tipo B | 2.29 | por cualquier otro medio: certificado, hoja de datos, resolución | no es "menos confiable" que Tipo A |
| Incertidumbre estándar | 2.30 | expresada como desviación estándar | es la que se combina |
| Incertidumbre expandida | 2.35 | $U = k\,u_c$ | es la que se informa |
| Factor de cobertura | 2.38 | el $k$ | $k = 2$ da ~95 % solo si hay suficientes grados de libertad |

El VIM en español (edición del CEM) dice "incertidumbre"; "incerteza" es el uso habitual en Argentina. Son lo mismo.

---

## 2. Error, incerteza y los tres ejes

**Error vs incerteza.** El error es una propiedad del resultado respecto de la realidad; la incerteza es una propiedad de lo que sé. Un resultado puede tener error chico e incerteza grande (hubo suerte) o error grande e incerteza chica (hay un sistemático que no conozco).

Hay tres clasificaciones **independientes** entre sí. Mezclarlas es el error conceptual más común.

| Eje | Opciones | Depende de |
|---|---|---|
| Comportamiento del efecto | aleatorio / sistemático | la física del efecto |
| Método de evaluación | Tipo A / Tipo B | de dónde saqué el número de $u$ |
| Tipo de medición | directa / indirecta | el modelo: ¿el resultado se calcula a partir de otras magnitudes? |

Los dos primeros se cruzan en cualquier combinación. Ejemplos con una IMU:

| | Evaluado Tipo A | Evaluado Tipo B |
|---|---|---|
| **Efecto aleatorio** | σ de muchas lecturas con el sensor quieto | error de cuantización: el paso del ADC es dato; se modela uniforme |
| **Efecto sistemático** | offset estimado promediando mediciones contra una referencia | offset tomado de la tolerancia de la hoja de datos; se modela uniforme |

> Regla práctica: "¿A o B?" se responde mirando **cómo obtuve el número**, no qué tipo de efecto es.

---

## 3. Promediar N lecturas

### 3.1 Por qué $\sqrt N$

Con $X_1, \dots, X_N$ de igual σ:

$$
\operatorname{Var}(\bar X)
= \frac{1}{N^2}\operatorname{Var}\Big(\sum_i X_i\Big)
= \frac{1}{N^2}\Big(\sum_i \operatorname{Var}(X_i) + \sum_{i\ne j}\operatorname{Cov}(X_i,X_j)\Big)
$$

Si son **independientes**, las covarianzas son 0 y queda

$$
\operatorname{Var}(\bar X) = \frac{\sigma^2}{N}
\qquad\Longrightarrow\qquad
u(\bar X) = \frac{\sigma}{\sqrt N}
$$

Lo que se suma son las **varianzas**, no las σ. Por eso la dispersión de la suma crece como $\sqrt N$ y la de la media baja como $1/\sqrt N$.

**No confundir:**
- $\bar X$ estima el valor; $\sigma/\sqrt N$ es la incerteza de esa estimación.
- $\sigma$ es la dispersión de **una** lectura y no cambia con N. La que cambia es $\sigma/\sqrt N$.

### 3.2 Cuándo deja de valer

| Situación | Qué pasa |
|---|---|
| Muestras correlacionadas (señal filtrada, sobremuestreo) | $\operatorname{Var}(\bar X) > \sigma^2/N$: hay menos información independiente de la que parece |
| Ruido 1/f o random walk | la varianza muestral no converge: cuanto más largo el registro, más grande da |
| Deriva (temperatura, envejecimiento) | el valor cambia mientras se promedia |
| Error sistemático | está en todas las lecturas: promediar no lo toca |

Con correlación, en general:

$$
\operatorname{Var}(\bar X) = \frac{\sigma^2}{N}\left[1 + 2\sum_{k=1}^{N-1}\Big(1-\frac kN\Big)\rho_k\right]
$$

con $\rho_k$ la autocorrelación a distancia $k$. Para un proceso con $\rho_k = \rho^k$ (autorregresivo de orden 1) y N grande, el corchete tiende a $\frac{1+\rho}{1-\rho}$. Se define un **número efectivo de muestras** $N_\text{eff} = N\,\frac{1-\rho}{1+\rho}$. Por ejemplo, con $\rho = 0{,}5$: $N_\text{eff} = N/3$, y la σ de la media es $\sqrt3 \approx 1{,}7$ veces lo que da la fórmula ingenua.

La herramienta para saber **hasta dónde** sirve promediar un sensor real es la varianza de Allan → [[senales-ruido#6. Varianza de Allan]].

---

## 4. Propagación

### 4.1 Ley de propagación (primer orden)

Modelo $Y = f(X_1, \dots, X_n)$. Linealizando alrededor de las estimaciones $x_i$:

$$
u_c^2(y) = \sum_i c_i^2\,u^2(x_i) + 2\sum_{i<j} c_i\,c_j\,u(x_i,x_j),
\qquad
c_i = \left.\frac{\partial f}{\partial x_i}\right|_{x}
$$

- $c_i$, **coeficiente de sensibilidad**: cuánto se mueve Y por unidad de $X_i$. Se puede calcular, derivar numéricamente o medir (mover $X_i$ y mirar Y).
- $u(x_i, x_j) = r_{ij}\,u(x_i)\,u(x_j)$, con $r_{ij}$ el coeficiente de correlación. Aparece cuando dos entradas comparten una fuente: el mismo instrumento, la misma referencia, la misma temperatura.
- $c_i\,u(x_i)$ es la **contribución** de cada entrada. Tiene las unidades de Y y es lo que se compara.

### 4.2 Casos frecuentes (entradas independientes)

| Modelo | Incerteza |
|---|---|
| $Y = X_1 \pm X_2$ | $u_Y^2 = u_1^2 + u_2^2$ (también para la resta) |
| $Y = aX$ | $u_Y = \lvert a\rvert\,u_X$ |
| $Y = X_1 X_2$ o $X_1/X_2$ | $\left(\frac{u_Y}{Y}\right)^2 = \left(\frac{u_1}{X_1}\right)^2 + \left(\frac{u_2}{X_2}\right)^2$ |
| $Y = X^p$ | $\frac{u_Y}{\lvert Y\rvert} = \lvert p\rvert\,\frac{u_X}{\lvert X\rvert}$ |
| $Y = \prod X_i^{p_i}$ | $\left(\frac{u_Y}{Y}\right)^2 = \sum_i p_i^2 \left(\frac{u_i}{X_i}\right)^2$ |

En productos y cocientes se trabaja en incertezas **relativas** (%, ppm). Por ejemplo, una frecuencia medida como $f = N/\Delta t$ hereda la incerteza relativa de la base de tiempo con coeficiente 1: 20 ppm en el reloj son 20 ppm en $f$.

### 4.3 Límites de la linealización y Monte Carlo

La fórmula de 4.1 falla cuando el término de segundo orden no es despreciable frente al primero:
- funciones muy curvas en la zona de trabajo: $\operatorname{atan}(x/z)$ con $z \to 0$;
- sensibilidad nula en el punto: $Y = X^2$ con $x \approx 0$ da $u_Y = 0$, que es absurdo;
- salida asimétrica: $\pm U$ deja de ser simétrico en probabilidad.

**Monte Carlo** (JCGM 101):
1. Asignar una distribución a cada entrada (y correlaciones, si hay).
2. Sortear $M$ juegos de entradas ($M \sim 10^6$) y evaluar $f$ en cada uno.
3. De la distribución de Y: media, desviación estándar e intervalo de cobertura por cuantiles (2,5 % y 97,5 % para 95 %).

Chequeo: en una zona donde $f$ es casi lineal, tiene que coincidir con 4.1.

---

## 5. Suma lineal o en cuadratura

Depende de la correlación entre las contribuciones:

$$
u^2(a + b) = u_a^2 + u_b^2 + 2r\,u_a u_b
\qquad
\begin{cases}
r = 0: & u = \sqrt{u_a^2 + u_b^2} \quad\text{(cuadratura)}\\
r = 1: & u = u_a + u_b \quad\text{(lineal)}
\end{cases}
$$

- **Independientes** (ruido de lectura, variaciones que no se repiten): cuadratura. $n$ contribuciones iguales dan $\sqrt n\,u$.
- **Totalmente correlacionadas** (el mismo efecto se repite en cada término): lineal. $n$ contribuciones iguales dan $n\,u$.

Ejemplo: sumar varias resistencias medidas con el mismo multímetro. El ruido de lectura se suma en cuadratura; el error de ganancia del multímetro es el mismo en todas y se suma linealmente. Para $n$ grande domina el correlacionado. Además, repetir las mediciones reduce solo la parte aleatoria.

---

## 6. Presupuesto GUM

### 6.1 Procedimiento (GUM, cap. 8)

1. **Modelo** $Y = f(X_i)$, incluyendo las correcciones como entradas (con valor 0 si no se conoce su signo, pero con incerteza).
2. **Estimación** de cada $x_i$ y su $u(x_i)$, por Tipo A o Tipo B.
3. **Coeficientes de sensibilidad** $c_i$ y covarianzas.
4. **Incerteza combinada** $u_c(y)$ (sección 4).
5. **Grados de libertad efectivos** y **factor de cobertura** $k$.
6. **Informar** $y \pm U$, con $k$ y la probabilidad de cobertura.

### 6.2 Tabla

| Magnitud $X_i$ | Estimación | $u(x_i)$ | Tipo / distribución | $c_i$ | Contribución $\lvert c_i\rvert\,u(x_i)$ | $\nu_i$ |
|---|---|---|---|---|---|---|
| … | | | | | | |
| **Combinada** | | | | | $u_c$ | $\nu_\text{eff}$ |

Se ordena por contribución. La fila más grande dice qué mejorar: reducir una contribución chica casi no mueve el total, porque se suman cuadrados.

### 6.3 Del dato al Tipo B

| Información disponible | Distribución | $u$ |
|---|---|---|
| Certificado: $U$ con factor $k$ | normal | $U/k$ |
| Tolerancia o especificación $\pm a$, sin más información | uniforme | $a/\sqrt3$ |
| Resolución $\delta$ de un indicador digital | uniforme de semiancho $\delta/2$ | $\delta/\sqrt{12}$ |
| $\pm a$, con los valores centrales más probables | triangular | $a/\sqrt6$ |
| Magnitud que oscila senoidalmente entre $\pm a$ | arcoseno | $a/\sqrt2$ |

Ejemplo: un multímetro indica 100,00 Ω con especificación ±(0,05 % de lectura + 2 dígitos). $a = 0{,}05 + 0{,}02 = 0{,}07$ Ω y $u = 0{,}07/\sqrt3 \approx 0{,}04$ Ω.

Tomar la tolerancia $\pm a$ como si fuera σ sobreestima $u$ en un 73 %. Tomarla como $2\sigma$ la subestima.

### 6.4 Grados de libertad y factor de cobertura

- Tipo A con $N$ lecturas: $\nu = N - 1$.
- Tipo B (GUM G.4.2): $\nu \approx \frac12\left(\frac{\Delta u}{u}\right)^{-2}$, donde $\Delta u/u$ es cuánto confío en mi propia $u$. Al 25 % da $\nu = 8$; si la tomo como exacta, $\nu = \infty$.
- **Welch–Satterthwaite:**

$$
\nu_\text{eff} = \frac{u_c^4}{\displaystyle\sum_i \frac{(c_i u_i)^4}{\nu_i}}
$$

- $k$ = cuantil de la t de Student con $\nu_\text{eff}$ para la probabilidad elegida. Con $\nu_\text{eff}$ grande, $k = 2$ da ~95 %.
- Si domina una contribución uniforme, la combinada no es normal y $k = 2$ sobrecubre: el 95 % de una uniforme está en $\pm 1{,}65\sigma$.

### 6.5 Cómo se informa

- $U$ con **una o dos cifras significativas**, y el resultado redondeado a la misma posición decimal.
- Decir $k$ y la probabilidad de cobertura: "$R = 100{,}02\ \Omega \pm 0{,}08\ \Omega$ ($k = 2$, ~95 %)".
- El presupuesto (la tabla) acompaña al resultado: sin él, el número no se puede auditar.

---

## 7. Resumen

| Idea | Ecuación |
|---|---|
| La incerteza es una σ, no un rango seguro | $u$ = desviación estándar de los valores atribuibles |
| Promediar ruido independiente | $u(\bar X) = \sigma/\sqrt N$ |
| Con correlación, hay menos muestras efectivas | $N_\text{eff} = N\,\frac{1-\rho}{1+\rho}$ (AR(1)) |
| Propagación a primer orden | $u_c^2 = \sum c_i^2 u_i^2 + 2\sum c_i c_j u_{ij}$ |
| Productos y cocientes: en relativo | $(u_Y/Y)^2 = \sum p_i^2 (u_i/X_i)^2$ |
| Correlacionados suman lineal; independientes, en cuadratura | $r = 1$: $u_a + u_b$; $r = 0$: $\sqrt{u_a^2 + u_b^2}$ |
| Tolerancia $\pm a$ sin más información | $u = a/\sqrt3$ |
| Lo que se informa | $U = k\,u_c$ |

---

## 8. Preguntas de repaso (sin mirar)

1. ¿Cuál es la diferencia entre error e incerteza? ¿Cuál de los dos se puede calcular?
2. Dá un ejemplo de efecto sistemático evaluado Tipo A y uno de efecto aleatorio evaluado Tipo B.
3. ¿En qué paso de la deducción de $\sigma/\sqrt N$ se usa la independencia?
4. Un sensor tiene ruido blanco y además deriva con la temperatura. ¿Qué pasa con la incerteza de la media a medida que promediás más tiempo?
5. ¿Cuándo se suman dos contribuciones linealmente y cuándo en cuadratura?
6. Una hoja de datos dice "±2 %". ¿Qué $u$ usás y qué supuesto hacés?
7. ¿Por qué mejorar la segunda fila de un presupuesto casi no cambia el total?
8. ¿Cuándo no alcanza la propagación lineal y qué hacés en su lugar?

---

## Fuentes

- JCGM 200:2012, *International vocabulary of metrology* (VIM), 3.ª ed.: 2.3, 2.13–2.19, 2.26, 2.28–2.30, 2.35, 2.38. Gratis en BIPM; versión en español del CEM.
- JCGM 100:2008, *Evaluation of measurement data — Guide to the expression of uncertainty in measurement* (GUM): §3 (conceptos), §4.2 (Tipo A), §4.3 (Tipo B; 4.3.7 uniforme, 4.3.9 triangular y trapezoidal), §5.1–5.2 (propagación, con y sin correlación), §6 (expandida), §7 (cómo informar), §8 (procedimiento), anexo G (grados de libertad, Welch–Satterthwaite).
- JCGM 101:2008, Suplemento 1 del GUM: propagación de distribuciones por Monte Carlo.
- J. R. Taylor, *An Introduction to Error Analysis*, 2.ª ed.: cap. 3 (propagación), cap. 4 (análisis estadístico, error de la media), cap. 5 (distribución normal).
- EA-4/02 M:2022, *Evaluation of the Uncertainty of Measurement in Calibration*: forma de la tabla y ejemplos resueltos.
- W. J. Riley, *Handbook of Frequency Stability Analysis*, NIST SP 1065: "Standard Variance" (por qué la varianza común no converge con ruido de baja frecuencia).
