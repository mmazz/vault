# Resumen

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

## Operaciones de bits

### AND

Se puede usar como **máscara** para seleccionar determinados bits.

Ejemplo:

$$
1101\ \text{AND}\ 0011 = 0001
$$

La máscara `0011` conserva los últimos dos bits y pone los demás en cero.

### XOR

Se puede usar para **invertir (flippear)** los bits seleccionados por una máscara.

$$
1101\ \text{XOR}\ 0011 = 1110
$$

Los bits donde la máscara vale `1` se invierten.

Además:

$$
x\ \text{XOR}\ x = 0
$$

Por ejemplo:

$$
1101\ \text{XOR}\ 1101 = 0000
$$
