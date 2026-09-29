# Probabilidad y distribuciones

Las herramientas que usan [[mediciones-incertezas]] y [[senales-ruido]]: esperanza y varianza, las distribuciones que aparecen al medir, cómo se estima a partir de muestras y cómo se ajusta un modelo.

---

## Esperanza y varianza

### Densidad

Para una variable continua, $f_X(x)$ es una **densidad**: la probabilidad de caer en un intervalo es el área bajo la curva.

$$
P(a \le X \le b) = \int_a^b f_X(x)\,dx,
\qquad
f_X(x) \ge 0,
\qquad
\int_{-\infty}^{\infty} f_X(x)\,dx = 1
$$

La última condición es la **normalización**. Una densidad propuesta "a ojo" (por ejemplo, constante en un intervalo) primero se normaliza: la constante sale de pedir que el área sea 1.

### Esperanza

$$
\mu = E[X] = \int_{-\infty}^{\infty} x\,f_X(x)\,dx
$$

Es una integral **definida**: el resultado es un **número**, no una función de $x$.

**LOTUS** (*law of the unconscious statistician*): para calcular la esperanza de una función de $X$ no hace falta la densidad de esa función.

$$
E[g(X)] = \int_{-\infty}^{\infty} g(x)\,f_X(x)\,dx
$$

### Varianza

$$
\operatorname{Var}(X) = \sigma^2 = E\big[(X-\mu)^2\big] = \int_{-\infty}^{\infty} (x-\mu)^2 f_X(x)\,dx
$$

$\sigma = \sqrt{\operatorname{Var}(X)}$ tiene las mismas unidades que $X$; la varianza, las unidades al cuadrado.

### Propiedades

| Propiedad | Vale | Comentario |
|---|---|---|
| $E[aX + bY + c] = a\,E[X] + b\,E[Y] + c$ | siempre | linealidad: no requiere independencia |
| $\operatorname{Var}(aX + b) = a^2\operatorname{Var}(X)$ | siempre | trasladar no cambia la dispersión; escalar la multiplica |
| $\operatorname{Var}(X) = E[X^2] - \mu^2$ | siempre | útil en papel; mala numéricamente si $\mu \gg \sigma$ (resta de dos números casi iguales) |
| $\operatorname{Cov}(X,Y) = E[(X-\mu_X)(Y-\mu_Y)] = E[XY] - \mu_X\mu_Y$ | siempre | mide cuánto varían juntas |
| $\operatorname{Var}(X \pm Y) = \operatorname{Var}(X) + \operatorname{Var}(Y) \pm 2\operatorname{Cov}(X,Y)$ | siempre | |
| $E[XY] = E[X]\,E[Y]$, $\operatorname{Cov}(X,Y) = 0$ | si son independientes | la covarianza nula no implica independencia |

Consecuencia de las dos últimas filas: **restar dos mediciones independientes no cancela su ruido, lo suma** ($\sigma^2_{X-Y} = \sigma_X^2 + \sigma_Y^2$). Lo que se cancela al restar es la parte **común**, la que está correlacionada entre las dos. En eso se basan las mediciones diferenciales.

### Analogía mecánica

Una densidad es una distribución de masa sobre una varilla, con masa total 1.

| Probabilidad | Mecánica |
|---|---|
| $f_X(x)$ | densidad lineal de masa |
| normalización, $\int f = 1$ | masa total 1 |
| $\mu$ | centro de masa |
| $\operatorname{Var}(X)$ | momento de inercia respecto del centro de masa |
| $\sigma$ | radio de giro |
| $E[X^2]$ | momento de inercia respecto del origen |
| $E[X^2] = \operatorname{Var}(X) + \mu^2$ | teorema de Steiner |
| $\operatorname{Var}(aX+b) = a^2\operatorname{Var}(X)$ | trasladar la varilla no cambia su momento; estirarla por $a$ lo multiplica por $a^2$ |
| uniforme de ancho $L$: $\sigma^2 = L^2/12$ | varilla homogénea: $I_{CM} = ML^2/12$ |

---

## Distribuciones

### Poisson

Distribución **discreta**. Modela la probabilidad de observar exactamente `k` eventos en un intervalo cuando el número esperado de eventos en ese intervalo es `λ`.

$$
P(X=k)=\frac{e^{-\lambda}\lambda^k}{k!}
$$

- `k`: número de eventos observados.
- `λ`: número esperado (medio) de eventos en el intervalo.
- No confundir `λ` con una probabilidad.

---

### Gaussiana o Normal

Distribución **continua**, caracterizada por una media `μ` y una desviación estándar `σ`.

$$
f_X(x)=
\frac{1}{\sigma\sqrt{2\pi}}
e^{-\frac{(x-\mu)^2}{2\sigma^2}}
$$

- `μ`: media, determina el centro.
- `σ`: desviación estándar, determina el ancho.
- `σ²`: varianza.

Para una variable continua, `f(x)` es una **densidad de probabilidad**, no la probabilidad de obtener exactamente `x`.

---

### Uniforme

Distribución **continua** en la que todos los valores dentro del intervalo tienen la misma densidad.

$$
f_X(x)=
\begin{cases}
\frac{1}{b-a}, & a\le x\le b \\
0, & \text{en otro caso}
\end{cases}
$$

Para una uniforme simétrica:

$$
X\sim U[-a,a]
$$

la media es

$$
E[X]=0
$$

la varianza es

$$
\operatorname{Var}(X)=\frac{a^2}{3}
$$

y la desviación estándar es

$$
\sigma=\frac{a}{\sqrt{3}}
$$

La varianza crece cuadráticamente con la escala, mientras que la desviación estándar crece linealmente.

#### Error de cuantización

Si dos niveles consecutivos están separados por `Δ` y se redondea al nivel más cercano, el error de cuantización se puede modelar, bajo ciertas condiciones, como:

$$
e\sim U\left[-\frac{\Delta}{2},\frac{\Delta}{2}\right]
$$

Por lo tanto:

$$
\operatorname{Var}(e)=\frac{\Delta^2}{12}
$$

$$
\sigma_e=\frac{\Delta}{\sqrt{12}}
$$

---

### Binomial

Distribución **discreta**. Da la probabilidad de obtener exactamente `k` éxitos en `n` ensayos.

Los ensayos se suponen independientes y cada uno tiene la misma probabilidad de éxito `p`.

$$
P(X=k)=
\binom{n}{k}
p^k(1-p)^{n-k}
$$

donde:

$$
\binom{n}{k}
=
\frac{n!}{k!(n-k)!}
$$

El término combinatorio cuenta de cuántas maneras pueden ubicarse los `k` éxitos entre los `n` ensayos.

---

### Triangular

Distribución **continua** con forma de triángulo. Para la simétrica de semiancho $a$ centrada en $c$:

$$
f_X(x) = \frac{a - \lvert x - c\rvert}{a^2}, \qquad \lvert x - c\rvert \le a
$$

$$
E[X] = c, \qquad \operatorname{Var}(X) = \frac{a^2}{6}
$$

Se usa cuando se sabe que el valor está en $\pm a$ y que los valores centrales son más probables que los extremos. Aparece también como suma de dos uniformes independientes del mismo ancho. La **trapezoidal** es el caso de dos uniformes de anchos distintos: con semiancho $a$ y techo de semiancho $\beta a$, $\operatorname{Var} = a^2(1+\beta^2)/6$.

---

### Exponencial

Distribución **continua** del tiempo entre eventos de un proceso de Poisson de tasa $\lambda$ (eventos por unidad de tiempo).

$$
f_T(t) = \lambda e^{-\lambda t}, \quad t \ge 0,
\qquad
E[T] = \frac1\lambda, \qquad \operatorname{Var}(T) = \frac1{\lambda^2}
$$

- **Sin memoria:** la probabilidad de esperar $t$ más no depende de cuánto se esperó ya.
- Si la tasa de fallas es constante, el tiempo medio entre fallas (MTBF) es $1/\lambda$.

---

### t de Student

Distribución **continua** de

$$
T = \frac{\bar X - \mu}{s/\sqrt N}
$$

cuando las $X_i$ son normales y σ se **estima** con la desviación estándar muestral $s$. Tiene $\nu = N - 1$ grados de libertad.

- Tiene colas más pesadas que la normal porque $s$ también fluctúa. Con $\nu \to \infty$ tiende a la normal.
- Cuantil para 95 % bilateral: 2,26 con $\nu = 9$; 2,09 con $\nu = 19$; 1,96 con $\nu = \infty$.
- $E[T] = 0$ (para $\nu > 1$); $\operatorname{Var}(T) = \nu/(\nu-2)$ (para $\nu > 2$).

---

### χ² (chi cuadrado)

Distribución **continua** de la suma de los cuadrados de $k$ normales estándar independientes.

$$
Q = \sum_{i=1}^{k} Z_i^2 \sim \chi^2_k,
\qquad
E[Q] = k, \qquad \operatorname{Var}(Q) = 2k
$$

Aparece en tres lugares:
- **Dispersión de la varianza muestral:** $(N-1)\,s^2/\sigma^2 \sim \chi^2_{N-1}$. Da intervalos de confianza para σ.
- **Bondad de ajuste:** la suma de residuos normalizados al cuadrado de un ajuste (sección de ajustes).
- **Intervalo de confianza de la varianza de Allan**, con grados de libertad equivalentes → [[senales-ruido#6. Varianza de Allan]].

---

### Resumen de momentos

| Distribución | Tipo | Parámetros | Media | Varianza |
|---|---|---|---|---|
| Poisson | discreta | $\lambda$ | $\lambda$ | $\lambda$ |
| Binomial | discreta | $n$, $p$ | $np$ | $np(1-p)$ |
| Normal | continua | $\mu$, $\sigma$ | $\mu$ | $\sigma^2$ |
| Uniforme | continua | $[a, b]$ | $(a+b)/2$ | $(b-a)^2/12$ |
| Triangular simétrica | continua | centro $c$, semiancho $a$ | $c$ | $a^2/6$ |
| Exponencial | continua | tasa $\lambda$ | $1/\lambda$ | $1/\lambda^2$ |
| t de Student | continua | $\nu$ | 0 | $\nu/(\nu-2)$ |
| χ² | continua | $k$ | $k$ | $2k$ |

Dos consecuencias prácticas:
- **Poisson:** si se cuentan $n$ eventos, la incerteza del conteo es $\approx \sqrt n$. La incerteza relativa baja como $1/\sqrt n$.
- **Binomial con cero fallas:** si en $n$ ensayos no hubo ninguna falla, la cota superior al 95 % de la probabilidad de falla es $\approx 3/n$ ("regla del tres"). Para afirmar "menos de 1 %" hacen falta ~300 ensayos sin fallas.

---

## Suma de variables y teorema central del límite

**Densidad de una suma.** Si $X$ e $Y$ son independientes, la densidad de $S = X + Y$ es la **convolución**:

$$
f_S(s) = \int_{-\infty}^{\infty} f_X(x)\,f_Y(s - x)\,dx
$$

Geométricamente: se desliza una densidad sobre la otra y se mide el área superpuesta.

**Teorema central del límite.** La suma de muchas variables independientes con varianza finita tiende a una normal, **cualquiera sea** la distribución de cada una. Con uniformes, la convergencia es rápida: la suma de cuatro ya se parece a una campana.

Consecuencias:
- Por eso muchos ruidos físicos son aproximadamente normales: son suma de muchas contribuciones chicas.
- Por eso una incerteza combinada de varias fuentes suele ser aproximadamente normal y $k = 2$ da ~95 %.
- **Falla** cuando una sola contribución domina: la suma se parece a esa contribución, no a una normal.

---

## Estimación a partir de muestras

Con $N$ muestras independientes de una misma distribución:

| Estimador | Fórmula | Qué estima |
|---|---|---|
| Media muestral | $\bar x = \frac1N\sum x_i$ | $\mu$ |
| Varianza muestral | $s^2 = \frac{1}{N-1}\sum (x_i - \bar x)^2$ | $\sigma^2$ |
| Desviación estándar de la media | $s/\sqrt N$ | la incerteza de $\bar x$ → [[mediciones-incertezas#3. Promediar N lecturas]] |

**Por qué $N-1$.** Los desvíos se miden respecto de $\bar x$, que se calculó con los mismos datos y queda más cerca de ellos que $\mu$. Por Steiner:

$$
\sum_i (x_i - \mu)^2 = \sum_i (x_i - \bar x)^2 + N(\bar x - \mu)^2
$$

Tomando esperanza, $N\sigma^2 = E\left[\sum (x_i - \bar x)^2\right] + \sigma^2$. Dividir por $N-1$ corrige ese sesgo. Se dice que $\bar x$ "gastó" un grado de libertad.

**La propia $s$ tiene incerteza.** Relativa, aproximadamente:

$$
\frac{u(s)}{s} \approx \frac{1}{\sqrt{2(N-1)}}
$$

Con $N = 10$ es ~24 %; con $N = 50$, ~10 %. Por eso con pocas muestras el intervalo se arma con la t de Student y no con la normal.

---

## Ajuste por cuadrados mínimos

### Cuadrados mínimos = máxima verosimilitud gaussiana

Modelo $y_i = f(x_i;\theta) + \varepsilon_i$, con $\varepsilon_i \sim N(0, \sigma_i^2)$ independientes. La verosimilitud de los parámetros $\theta$ es

$$
L(\theta) \propto \prod_i \exp\left(-\frac{\big(y_i - f(x_i;\theta)\big)^2}{2\sigma_i^2}\right)
\qquad\Longrightarrow\qquad
-2\ln L = \chi^2(\theta) + \text{cte},
\qquad
\chi^2(\theta) = \sum_i \left(\frac{y_i - f(x_i;\theta)}{\sigma_i}\right)^2
$$

Maximizar $L$ es minimizar $\chi^2$. Los **supuestos** (errores gaussianos, independientes, con σ conocidas) son lo que hay que verificar. Si las $\sigma_i$ son distintas, el ajuste es **ponderado** con pesos $1/\sigma_i^2$.

### Modelo lineal en los parámetros

$y = A\theta + \varepsilon$, con $A$ la matriz de diseño y $W = \operatorname{diag}(1/\sigma_i^2)$:

$$
\hat\theta = (A^\top W A)^{-1} A^\top W\,y,
\qquad
\operatorname{Cov}(\hat\theta) = (A^\top W A)^{-1}
$$

- "Lineal" es en los **parámetros**: $y = a + bx + cx^2$ es lineal; $y = a\,e^{-bx}$ no.
- La **matriz de covarianza** de los parámetros sale del ajuste. La diagonal da las incertezas; fuera de la diagonal, qué parámetros están correlacionados. Esa correlación hay que propagarla si después se usan los parámetros juntos.
- Si las $\sigma_i$ no se conocen y se estiman de los residuos, se escala $\operatorname{Cov}(\hat\theta)$ por $\chi^2/\nu$.

### Diagnóstico

- **χ² reducido:** $\chi^2/\nu$, con $\nu = N - p$ (datos menos parámetros). Debería dar $\approx 1 \pm \sqrt{2/\nu}$.
  - Mucho mayor que 1: el modelo no alcanza o las σ están subestimadas.
  - Mucho menor que 1: las σ están sobreestimadas.
- **Residuos:** graficarlos siempre, contra $x$ y contra cualquier otra variable que pueda influir (tiempo, temperatura). Si tienen estructura, el modelo está incompleto. Es más informativo que el χ².
- **Sobreajuste:** agregar parámetros siempre baja χ². Un término nuevo se justifica si la mejora es significativa y tiene sentido físico.

---

## Preguntas de repaso (sin mirar)

1. ¿Por qué hay que normalizar una densidad antes de calcular su media? ¿Qué error aparece si no se hace?
2. ¿Por qué $E[X]$ es un número y no una función de $x$?
3. ¿Qué teorema de mecánica es $E[X^2] = \operatorname{Var}(X) + \mu^2$?
4. Restás dos mediciones independientes con el mismo ruido. ¿Qué pasa con la σ? ¿Qué sí se cancela al restar?
5. ¿Por qué el teorema central del límite justifica usar $k = 2$? ¿Cuándo no lo justifica?
6. ¿Por qué se divide por $N - 1$? ¿Cuánto vale la incerteza relativa de $s$ con 10 muestras?
7. Contaste 0 fallas en 100 ensayos. ¿Qué podés decir de la tasa de fallas?
8. ¿Qué supuestos convierten cuadrados mínimos en máxima verosimilitud? ¿Qué mirás para saber si el ajuste es bueno?

---

## Fuentes

- JCGM 100:2008 (GUM): §4.3.7 (uniforme), §4.3.9 (triangular y trapezoidal), anexo C (conceptos de probabilidad), E.4.3 (incerteza de la desviación estándar), anexo G (t de Student).
- J. R. Taylor, *An Introduction to Error Analysis*, 2.ª ed.: cap. 5 (normal), cap. 8 (cuadrados mínimos), cap. 10 (binomial), cap. 11 (Poisson), cap. 12 (test de χ²).
- NIST/SEMATECH *e-Handbook of Statistical Methods*: §1.3.6 (distribuciones), §1.3.5 (tests cuantitativos).
- S. W. Smith, *The Scientist and Engineer's Guide to Digital Signal Processing*: cap. 2 (estadística, pdf, histograma).
