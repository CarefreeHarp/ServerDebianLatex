# Identificación de artículos seminales

**Proyecto:** Sistema híbrido de predicción y visualización de incendios forestales
(FARSITE + WFDS, SIG, HPC y biblioteca de escenarios precalculados)
**Entregable:** BOI — Búsqueda y Organización de Información
**Motores consultados:** IEEE Xplore, ACM Digital Library, JSTOR, Scopus, CiteSeerX, Google Académico

---

## 0. Nota metodológica sobre los motores

| Motor | Estado | Uso efectivo |
|---|---|---|
| **IEEE Xplore** | Operativo | Ejes de HPC, *surrogate models* y vecino más cercano. Aporta *Author Keywords*, *IEEE Keywords* e *Index Terms* (indización controlada) y el contador «Cited by: Papers (N)». |
| **ACM Digital Library** | Operativo | Simulación paralela y distribuida, asimilación de datos, estructuras de búsqueda. Aporta conteo de citas y términos CCS. |
| **JSTOR** | Operativo | Eje WUI: literatura de *Ecological Applications* y *PNAS* (Radeloff, Calkin, Spyratos, Alexandre). |
| **Google Académico** | Consultado al inicio y luego descartado por indicación explícita | De ahí provienen los conteos de citación de Rothermel 1972, Finney 1998, Mell 2007 y Papadopoulos 2011 que se reportan abajo. |
| **CiteSeerX** | **No disponible** | El dominio `citeseerx.ist.psu.edu` redirige a un archivo del Internet Archive y las rutas de búsqueda devuelven 404. El servicio está retirado. |
| **Scopus** | **No disponible sin sesión institucional** | La pestaña no responde a navegación ni a extracción; requiere autenticación de la universidad. Recomendación: repetir las búsquedas desde la VPN/proxy de la Javeriana para obtener los *Indexed Keywords* oficiales de Scopus. |

Las palabras clave que dieron mejores resultados fueron, por eje:
`wildfire spread simulation FARSITE` · `Rothermel fire spread model` · `wildland-urban interface fire` ·
`parallel high performance computing wildfire simulation` · `wildfire spread prediction deep learning surrogate` ·
`approximate nearest neighbor search` · `data assimilation wildfire spread simulation`.

---

## 1. Artículo seminal principal

> **Finney, M. A. (1998, rev. 2004). *FARSITE: Fire Area Simulator — Model Development and Evaluation*. Research Paper RMRS-RP-4. USDA Forest Service, Rocky Mountain Research Station. 47 p.**
> Citas: **2.379** (Google Académico). Clave BibTeX: `finney1998farsite`.

**Por qué es el seminal principal de *este* proyecto.** El proyecto no propone un modelo físico nuevo: propone una capa de integración sobre dos simuladores existentes. El artefacto sobre el que se apoya toda la mitad «escala forestal» del sistema —y del que dependen el formato de salida, la geometría del perímetro y los parámetros que el adaptador tendrá que traducir a WFDS— se define aquí.

Tres razones concretas:

1. **Es el punto de inflexión que convierte un modelo 1-D en un sistema espacial.** Finney integra los modelos preexistentes (superficie, copas, focos secundarios, aceleración, humedad del combustible) mediante *vector propagation* del perímetro, controlando resolución espacial y temporal. Es el paso de «¿a qué velocidad avanza el frente?» a «¿qué polígono habrá quemado a las 14:00?».
2. **Define exactamente el objeto que el pipeline debe interpretar.** El modelo produce perímetros vectoriales (polígonos) a intervalos especificados, cuyos vértices llevan tasa de propagación e intensidad y se interpolan a rásteres. Ese es literalmente el insumo del bloque «Interpretación → Transformación → Generación» del proyecto.
3. **Es la referencia obligada de la literatura operativa.** Papadopoulos y Pavlidou (2011), tras revisar 23 simuladores, concluyen que FARSITE es el que «destaca sobre el resto» y lo evalúan en profundidad.

*Candidato alternativo considerado:* Rothermel (1972) tiene el doble de citas, pero pertenece al eje «Modelo Rothermel» y describe un modelo puntual, no un sistema de simulación espacial. Asignarlo al eje que le corresponde y dejar Finney como seminal principal evita duplicar y refleja mejor el aporte del proyecto.

---

## 2. Seminal por eje temático

### 2.1 Modelo Rothermel

> **Rothermel, R. C. (1972). *A Mathematical Model for Predicting Fire Spread in Wildland Fuels*. Research Paper INT-115. USDA Forest Service.**
> Citas: **4.879** (Google Académico). Clave: `rothermel1972wildland`.

Es el artículo más citado de toda la bibliografía reunida y el que sostiene, directa o indirectamente, casi todo lo demás. Su aporte irrepetible: *«El modelo es completo en el sentido de que no se requiere conocimiento previo de las características de combustión de un combustible»* — sólo entradas físicas y químicas medibles en campo más viento y pendiente. Introduce además el concepto de **modelo de combustible**, que es lo que permite parametrizar un terreno real sin volver a medirlo.

Su condición de seminal se comprueba por descendencia: Albini (1976) construye los nomogramas sobre él; Anderson (1982) y Scott & Burgan (2005) tabulan sus modelos de combustible; FARSITE lo usa como modelo de superficie; Smith et al. (2016) paralelizan *sus ecuaciones* en GPU. Sin Rothermel 1972 no hay FARSITE, y sin FARSITE no hay proyecto.

**Cadena de referencia recomendada para el marco teórico:**
`rothermel1972wildland` → `albini1976estimating` → `anderson1982aids` → `vanwagner1977crown` → `rothermel1991crown` → `scott2005standard`.

---

### 2.2 HPC aplicado a la paralelización de simulaciones

> **Fujimoto, R. M. (1990). Parallel Discrete Event Simulation. *Communications of the ACM*, 33(10), 30–53.**
> Citas: **1.549** (ACM Digital Library). Clave: `fujimoto1990parallel`.

Es el artículo fundacional del campo: define el problema de ejecutar *una sola* simulación de eventos discretos sobre un computador paralelo y establece la taxonomía (protocolos conservadores vs. optimistas, *Time Warp*) que sigue vigente. Su observación central sigue describiendo el problema del proyecto: la simulación paralela «representa un dominio de problemas que a menudo contiene cantidades sustanciales de paralelismo y que, paradójicamente, resulta sorprendentemente difícil de paralelizar en la práctica».

**Punto de inflexión específico del dominio de incendios** (útil para justificar la arquitectura del clúster): la línea DDDAS del grupo de Cortés y Margalef (Universitat Autònoma de Barcelona) — `rodriguez2008adaptive` (10 artículos citantes en IEEE) y `rodriguez2010datainjection` — que introduce la idea de calibrar parámetros en paralelo bajo plazos de tiempo real, y su evolución a nube en `fraga2021urgent`. Para la vertiente GPU, `smith2016gpu` reporta aceleraciones de **64× a 229×** sobre las ecuaciones de Rothermel, y `canales2025edge` compara plataformas *edge* y nube con FARSITE acelerado en CUDA — el trabajo más cercano al problema concreto de generar miles de escenarios.

Complemento de revisión: `fujimoto2016research` (101 citas ACM) actualiza los retos abiertos, incluida la explotación de GPU y nube.

---

### 2.3 Fuego en la interfaz urbano-forestal (WUI)

> **Cohen, J. D. (2000). Preventing Disaster: Home Ignitability in the Wildland–Urban Interface. *Journal of Forestry*, 98(3), 15–21.**
> Clave: `cohen2000preventing`.

Es el artículo que cambia la pregunta. Antes de él, el desastre en la WUI se entendía como un problema de control del incendio; Cohen demuestra —con modelado, experimentos y estudios de caso— que «la ignitabilidad de una vivienda durante un incendio forestal depende de las características de la vivienda y de su entorno inmediato». De ahí nacen el concepto de *home ignition zone* y buena parte de la política pública posterior.

**Evidencia de seminalidad** (no se pudo obtener conteo de citas con los motores permitidos, así que se documenta por influencia trazable):
- Es la **referencia \[1\]** de la revisión de Caton et al. (2017), que a su vez acumula **216 citas** (Springer).
- Calkin et al. (2014, *PNAS*) reformulan su tesis casi literalmente: superar la percepción del desastre en la WUI «como un problema de control del incendio y no como un problema de ignición de la vivienda» reducirá la pérdida de viviendas.

**Seminales complementarios del eje, según qué se quiera sustentar:**

| Sub-pregunta | Referencia | Evidencia |
|---|---|---|
| ¿Qué es y dónde está la WUI? | `radeloff2005wui` (*Ecological Applications*) | Define y cartografía la WUI en EE.UU.; localizado vía JSTOR |
| ¿Cuánto está creciendo el riesgo? | `radeloff2018rapid` (*PNAS*) | **1.011 citas** (contador de PNAS) |
| ¿Por qué vías se propaga el fuego a las estructuras? | `caton2017pathways` (*Fire Technology*) | **216 citas** (Springer) |
| ¿Qué necesita la investigación en WUI? | `mell2010wui` (*IJWF*) | Documento-agenda del grupo NIST |

---

### 2.4 Acoplamiento entre FARSITE y WFDS

**Hallazgo importante: no existe un artículo seminal de acoplamiento directo FARSITE ↔ WFDS.** Las búsquedas en IEEE Xplore, ACM DL y JSTOR con combinaciones de `FARSITE`, `WFDS`, `coupling`, `multiscale` y `wildland-urban interface` no devuelven ningún trabajo que construya ese *pipeline*. **Ese vacío es, precisamente, el aporte del proyecto** y conviene declararlo así en la propuesta: no se está reimplementando algo existente.

Como seminal *proxy* del eje se propone el artículo que plantea explícitamente la necesidad de esa articulación multiescala:

> **Mell, W. E., Manzello, S. L., Maranghides, A., Butry, D. T. & Rehm, R. G. (2010). The Wildland–Urban Interface Fire Problem — Current Approaches and Research Needs. *International Journal of Wildland Fire*, 19(2), 238–251.**
> Clave: `mell2010wui`.

El artículo sostiene que hacen falta «experimentos de laboratorio, mediciones de campo y modelos de comportamiento del fuego» combinados para determinar las condiciones de exposición que enfrentan comunidades y estructuras — es decir, exactamente la división de trabajo entre un modelo de paisaje y un modelo físico que propone el proyecto.

**Seminal del extremo físico (WFDS como herramienta):**

> **Mell, W., Jenkins, M. A., Gould, J. & Cheney, P. (2007). A Physics-Based Approach to Modelling Grassland Fires. *International Journal of Wildland Fire*, 16(1), 1–22.**
> Citas: **852** (Google Académico). Clave: `mell2007physics`.

Es el artículo de referencia de WFDS y justifica el diseño híbrido con un argumento que el proyecto puede citar tal cual: los modelos físicos «requieren significativamente más recursos computacionales que los modelos de propagación más usados, que son semiempíricos o empíricos», pero existen problemas —y menciona en primer lugar los incendios en la interfaz urbano-forestal— «que quedan fuera del alcance de los modelos empíricos y semiempíricos».

**Referencias de apoyo para el diseño del adaptador:** `mell2009douglasfir` (WFDS a escala de árbol individual, la escala a la que trabajaría el módulo urbano), `linn2002firetec` y `coen2013wrffire` (otras experiencias de acoplamiento atmósfera–fuego, útiles como antecedentes de interoperabilidad).

---

### 2.5 Surrogate models

> **Sacks, J., Welch, W. J., Mitchell, T. J. & Wynn, H. P. (1989). Design and Analysis of Computer Experiments. *Statistical Science*, 4(4), 409–423.**
> Clave: `sacks1989design`.

Es el artículo que funda el área y, además, plantea el problema del proyecto en abstracto casi palabra por palabra: los códigos «son computacionalmente costosos de ejecutar, y un objetivo común de un experimento es ajustar un predictor más barato de la salida a los datos». La propuesta de modelar la salida determinista como realización de un proceso estocástico da la base estadística para **elegir qué escenarios simular** (diseño de experimentos) y para **cuantificar la incertidumbre de la respuesta recuperada** — que es justamente lo que la biblioteca de escenarios precalculados necesita para no devolver un escenario «parecido» sin decir cuán parecido.

**Aplicación directa al fuego (el estado del arte que el proyecto debe citar y superar o complementar):**

| Referencia | Aporte | Evidencia |
|---|---|---|
| `hodges2019wildland` (*Fire Technology*) | Red convolucional (DCIGN) que predice mapas de quema con **coste computacional tres órdenes de magnitud menor** | — |
| `cheng2023generative` (*IEEE TETCI*) | Autoencoder variacional 3-D que genera escenarios no vistos y entrena un *surrogate* de propagación | **16** artículos citantes (IEEE) |

**Nota de encuadre:** el proyecto usa una estrategia de *recuperación* (biblioteca + búsqueda del escenario más parecido) en lugar de una de *regresión* (red neuronal que predice el resultado). Conviene argumentar la elección frente a estos dos trabajos: la recuperación devuelve una simulación física real y auditable, no una interpolación aprendida.

---

## 3. Eje transversal no solicitado pero necesario: similaridad y recuperación

El bloque «¿cómo defino que dos escenarios son parecidos?» tiene su propio seminal, y es de los más citados de toda la bibliografía:

> **Cover, T. M. & Hart, P. E. (1967). Nearest Neighbor Pattern Classification. *IEEE Transactions on Information Theory*, 13(1), 21–27.**
> Citas: **14.077** artículos citantes (IEEE Xplore). Clave: `cover1967nearest`.

Fija la garantía teórica de la regla del vecino más cercano —su probabilidad de error está acotada por el doble de la de Bayes— y con ella la frase que resume el enfoque del proyecto: «la mitad de la información de clasificación de un conjunto infinito de muestras está contenida en el vecino más cercano».

Para la implementación:
- `bentley1975kdtree` — árboles k-d, indexación multidimensional clásica (búsqueda de vecino más cercano en *O*(log n) empírico).
- `malkov2020hnsw` — HNSW, el estado del arte en búsqueda aproximada; **1.399** artículos citantes (IEEE). Es la estructura que usaría el proyecto si la biblioteca crece a decenas de miles de escenarios con vectores de condiciones de alta dimensión.

---

## 4. Resumen ejecutivo

| Eje | Artículo seminal | Año | Evidencia de citación | Clave BibTeX |
|---|---|---|---|---|
| **Principal (proyecto)** | Finney — FARSITE: Fire Area Simulator | 1998 | 2.379 (G. Académico) | `finney1998farsite` |
| Modelo Rothermel | Rothermel — A Mathematical Model for Predicting Fire Spread… | 1972 | 4.879 (G. Académico) | `rothermel1972wildland` |
| HPC / paralelización | Fujimoto — Parallel Discrete Event Simulation | 1990 | 1.549 (ACM DL) | `fujimoto1990parallel` |
| Fuego en WUI | Cohen — Preventing Disaster: Home Ignitability in the WUI | 2000 | Ref. \[1\] de Caton 2017 (216 citas) | `cohen2000preventing` |
| Acoplamiento FARSITE–WFDS | *(vacío en la literatura)* → Mell et al. — The WUI Fire Problem | 2010 | Agenda de investigación NIST | `mell2010wui` |
| — extremo físico (WFDS) | Mell et al. — A Physics-Based Approach to Modelling Grassland Fires | 2007 | 852 (G. Académico) | `mell2007physics` |
| Surrogate models | Sacks et al. — Design and Analysis of Computer Experiments | 1989 | Fundacional del área | `sacks1989design` |
| *(transversal)* Similaridad / NN | Cover & Hart — Nearest Neighbor Pattern Classification | 1967 | 14.077 (IEEE Xplore) | `cover1967nearest` |


