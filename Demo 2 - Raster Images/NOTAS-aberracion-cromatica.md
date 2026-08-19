# Notas — aberración cromática

Documento de estudio. **El HTML no lo referencia**: bórralo antes del commit si no lo quieres publicar.

---

## 0. El puente con el Cap. 3

Hay que decirlo de frente: **la aberración cromática no es un tema del Cap. 3**.
Es óptica — pasa dentro del lente, antes de que exista un pixel.

Pero para *simularla* no queda otra que usar los tres conceptos centrales del capítulo:

| Lo que necesitas | Concepto del Cap. 3 |
|---|---|
| Procesar R, G y B por separado | representación RGB, canales independientes |
| Leer en coordenadas fraccionarias | reconstrucción / filtrado (bilineal) |
| Hacerlo en luz lineal, no en sRGB | corrección de gama |

Por eso este demo **amarra los otros dos** que se presentaron (antialiasing y gama)
en lugar de ser un cuarto tema suelto.

---

## 1. La causa física: dispersión

El índice de refracción de un material **depende de la longitud de onda**.
En el rango visible se aproxima muy bien con la **ecuación de Cauchy**:

$$n(\lambda) = A + \frac{B}{\lambda^2}$$

Como $\lambda$ está al cuadrado en el denominador, **el azul (λ corta) tiene índice
más alto que el rojo** → el azul se desvía más.

Coeficientes usados en el demo (valores estándar de tabla, $B$ en µm²):

| Vidrio | A | B | Comportamiento |
|---|---|---|---|
| BK7 (crown) | 1.5046 | 0.00420 | poca dispersión |
| BaF10 | 1.6700 | 0.00743 | media |
| SF10 (flint denso) | 1.7280 | 0.01342 | mucha dispersión |

Con λ_R = 610 nm, λ_G = 550 nm, λ_B = 465 nm, para BK7 sale:

```
n_R = 1.51589      n_G = 1.51848      n_B = 1.52402
```

**Comprobación de que el modelo no está inventado:** el demo calcula un número de
Abbe $V \approx (n_G - 1)/(n_B - n_R) = 63.7$, y el valor publicado del BK7 real es
**≈ 64.2**. Cuadra.

---

## 2. De índice a desplazamiento en píxeles

La cadena completa son tres pasos:

**Paso 1 — índice → distancia focal.** Ecuación del fabricante de lentes:

$$\frac{1}{f} = (n-1)\left(\frac{1}{R_1} - \frac{1}{R_2}\right) \quad\Longrightarrow\quad f \propto \frac{1}{n-1}$$

Como $n_B > n_R$, entonces $f_B < f_R$: **el azul enfoca más cerca que el rojo.**

**Paso 2 — distancia focal → magnificación.** Distinta $f$ ⇒ distinta magnificación.
En el demo se modela como un factor de escala por canal, relativo al verde:

$$s_c = 1 + E\cdot\left(\frac{f_c}{f_G} - 1\right)$$

donde $E$ es el factor de exageración (los lentes reales están corregidos y el
efecto es sutil; hay que amplificarlo para verlo en clase).

**Paso 3 — magnificación → remuestreo.** Para cada pixel de destino se lee el
canal desde una copia escalada distinta de la imagen fuente:

$$\text{out}_c(p) = \text{src}_c\!\left(\text{centro} + \frac{p - \text{centro}}{s_c}\right)$$

El desplazamiento resultante es:

$$|\Delta| = r\cdot\left(1 - \frac{1}{s_c}\right)$$

**Crece con el radio $r$ y vale exactamente cero en el centro óptico.** Por eso en
el demo las anillas pequeñas del centro quedan blancas por más que subas la exageración.

---

## 3. Los dos tipos de aberración

| | Longitudinal (axial) | Lateral (transversal) |
|---|---|---|
| Qué cambia | la **distancia** de enfoque | la **magnificación** |
| Dónde se ve | en todo el cuadro, parejo | crece hacia los bordes, cero en el centro |
| Aspecto | halos de color en zonas desenfocadas | franjas de color en los bordes |
| ¿Se simula en gráficas? | rara vez (necesita desenfoque por canal) | **sí, es la estándar** |

Este demo implementa la **lateral**.

---

## 4. Por qué las líneas radiales no fringean

Este es el mejor momento de la presentación.

El desplazamiento $\Delta$ apunta **radialmente** (hacia afuera desde el centro).
Un borde solo se ve desplazado si el movimiento lo cruza:

- **Borde tangencial** (perpendicular al radio) → el desplazamiento lo cruza de frente → **fringing máximo**
- **Borde radial** (paralelo al radio) → el desplazamiento corre *a lo largo* del borde → **no pasa nada**

Por eso la carta de prueba tiene arcos concéntricos a la izquierda y radios a la
derecha, **a los mismos radios**: la única diferencia es la orientación, y el
resultado es dramático. La vista **Δ** lo confirma numéricamente — los arcos se
prenden, los radios quedan oscuros.

(Las puntas de los radios sí fringean, porque una punta es un borde tangencial.)

---

## 5. El detalle de gama (conecta con tu compañero)

El botón `resample in: linear light / sRGB (wrong)`.

Un valor sRGB **no es proporcional a la luz** — guarda aproximadamente la potencia
$1/2.2$ de la intensidad. Interpolar entre dos píxeles es promediar, y promediar
solo tiene sentido físico sobre cantidades lineales:

```
correcto:    decodificar → interpolar → recodificar
incorrecto:  interpolar directamente sobre bytes sRGB
```

Hacerlo mal produce bordes que se ven **más oscuros y más saturados** de lo que
deberían. Es el mismo error que hace que un downscale de una imagen se vea sucio.

---

## 6. El detalle de remuestreo (conecta con el otro compañero)

Como $p/s_c$ casi nunca cae en un pixel entero, hay que **reconstruir** la señal.
El botón `reconstruction`:

- **nearest** → redondea al pixel más cercano. Los bordes se ven escalonados y el
  desplazamiento avanza a saltos de 1 pixel.
- **bilinear** → interpola entre los 4 vecinos. Desplazamiento subpixel suave.

Esto es literalmente el tema de reconstrucción del Cap. 3 / Cap. 10, aplicado.

---

## 7. Cómo se hace en la industria

En un juego es un *post-process* de una sola pasada, casi idéntico a esto:

```glsl
vec2 d = uv - 0.5;
float r = texture(tex, 0.5 + d * (1.0 + k)).r;
float g = texture(tex, 0.5 + d              ).g;
float b = texture(tex, 0.5 + d * (1.0 - k)).b;
```

Se usa como recurso **estético** (sensación de lente real, cámara barata,
distorsión bajo daño) más que como simulación física. Los lentes reales se
corrigen con **dobletes acromáticos**: se pega un crown y un flint cuyas
dispersiones se cancelan — por eso el demo tiene BK7 y SF10 como presets.

---

## 8. Guion sugerido de 20 minutos

1. **Arranca con la carta sin efecto** (exageración 0). Todo blanco y negro.
2. **Sube la exageración.** Aparecen las franjas. Pregunta: *¿por qué solo en unos bordes?*
3. **Señala el centro.** Las anillas pequeñas siguen blancas. `|Δ| = r(1 − 1/s)`, cero en r = 0.
4. **Señala izquierda vs derecha.** Arcos fringean, radios no. Explica el argumento
   del desplazamiento radial. Activa la vista **Δ** para rematarlo.
5. **Mueve la sonda a un borde** y muestra el perfil de canales: las tres curvas
   corridas entre sí. *Eso es la aberración, medida en píxeles.*
6. **Cambia el vidrio** de BK7 a SF10. Más dispersión → más separación. Menciona
   el número de Abbe y que cuadra con el valor real.
7. **Cierra con los dos botones que conectan** con los demos de tus compañeros:
   `sRGB (wrong)` y `nearest`. Ahí cierras el círculo del Cap. 3.

---

## 9. Chuleta

| Concepto | Fórmula | Control |
|---|---|---|
| Dispersión | $n(\lambda) = A + B/\lambda^2$ | `glass` |
| Focal | $f \propto 1/(n-1)$ | — |
| Escala por canal | $s_c = 1 + E(f_c/f_G - 1)$ | `exaggeration` |
| Remuestreo | $\text{out}_c(p) = \text{src}_c(c + (p-c)/s_c)$ | `reconstruction` |
| Desplazamiento | $\lvert\Delta\rvert = r(1 - 1/s_c)$ | se ve en `Δ` |
| Espacio de color | decodificar → operar → recodificar | `resample in` |
