# Diccionario de Variables 

Este documento describe el significado de cada variable incluida en el set
de datos, las preguntas de la encuesta detrás de las variables derivadas
y las decisiones metodológicas más relevantes para su correcta
interpretación.

---

## 1. Identificadores del Panel

| Variable | Descripción |
|----------|-------------|
| **`n`** | Número correlativo de observación (índice de fila). No tiene interpretación sustantiva. |
| **`id`** | Identificador único de cada celda del panel. Se construye concatenando el nombre de la región, el código de UBIGEO y el año (por ejemplo: `AMAZONAS_1_2024`). Es práctico para hacer cruces rápidos con otros datasets. |
| **`region`** | Nombre del departamento del Perú al que corresponde la observación (ej. *Lima*, *Arequipa*, *Áncash*). |
| **`ubigeo`** | Código numérico del departamento (1 al 25), equivalente a los dos primeros dígitos del UBIGEO oficial del INEI. Es la variable **transversal** (*cross-sectional*) del panel. |
| **`year`** | Año al que corresponde la observación. Es la variable **temporal** (*time-series*) del panel. Abarca el período 2004-2024. |

La estructura es, por tanto, un **panel balanceado** con 25 unidades
(departamentos) observadas durante 21 años.

---

## 2. Variables Principales

### 2.1 `mig_int` — Stock de migrantes internacionales

Número total de personas **nacidas fuera del Perú** que residen en cada
departamento en el año correspondiente. Sigue el estándar internacional
de Naciones Unidas, que define el stock por lugar de nacimiento (no por
nacionalidad ni por tiempo de residencia).

#### Metodología por período

Dado que las preguntas disponibles en las encuestas del INEI cambian a
lo largo del tiempo, la variable combina tres fuentes y enfoques:

| Período | Fuente | Identificación del migrante |
|---------|--------|------------------------------|
| **2007 y 2017** | Censos Nacionales de Población y Vivienda | Pregunta directa de país de nacimiento, aplicada al universo poblacional. |
| **2004-2006 y 2008-2016** | ENAHO — Módulo 02 | Combinación de respuestas a **P208A1** y **P208A2**. |
| **2018-2024** | ENAHO — Módulo 04 + ENPOVE 2018/2022 + ENAHOPV 2024 | Combinación de respuestas a **P401G1** y **P401G2**, empalmado con las encuestas dedicadas a la población venezolana. |

#### Identificación del migrante en 2004-2006 y 2008-2016

En ese período, el Módulo 02 de la ENAHO (Características de los
Miembros del Hogar) contenía dos preguntas sobre el lugar de nacimiento
de cada persona:

> **P208A1:** *¿Nació en este distrito?*

> **P208A2:** *¿En qué provincia y distrito nació?* (se registra el
> código de distrito).

Se clasifica a una persona como **migrante internacional** cuando
cumple, **simultáneamente**, las dos condiciones siguientes:

- **Condición sobre P208A1:** la persona respondió **"No"**, es decir,
  reporta el valor **`0`** en esa variable. Esto significa que no nació
  en el distrito donde reside actualmente.
- **Condición sobre P208A2:** el código de distrito de nacimiento
  registrado tiene sus **dos primeros dígitos fuera del rango `01`-`25`**,
  que son los códigos válidos de los 25 departamentos del Perú. Estos
  códigos "fuera de rango" (típicamente comenzando en `00`, por ejemplo
  `004037`, `004009`, `006022`) corresponden a la codificación que el
  INEI utiliza para países extranjeros.

El conteo se pondera con el factor de expansión poblacional de la
ENAHO (`FACPOB07`) y se agrega por departamento de residencia.

#### Identificación del migrante en 2018-2024

A partir de 2018 la ENAHO **elimina** las preguntas P208A1 y P208A2 del
Módulo 02. El bloque con información migratoria se traslada al Módulo
04 (Salud) y pasa a apoyarse en la siguiente pareja de preguntas:

> **P401G1:** *Cuando usted nació, ¿vivía su madre en este distrito?*

> **P401G2:** *¿En qué distrito y provincia vivía su madre?* (al momento
> del nacimiento de la persona).

Se clasifica a una persona como **migrante internacional** cuando
cumple, **simultáneamente**, las dos condiciones siguientes:

- **Condición sobre P401G1:** la persona respondió **"No"**, es decir,
  reporta el valor **`2`** en esa variable. Esto significa que su madre
  no vivía en el distrito actual al momento de su nacimiento.
- **Condición sobre P401G2:** el código de distrito donde vivía la
  madre tiene sus **dos primeros dígitos igual a `00`**. Este prefijo
  es el que el INEI reserva para identificar países extranjeros,
  quedando fuera del rango `01`-`25` correspondiente a los 25
  departamentos peruanos.

Como estas dos preguntas están en el Módulo 04, para agregar por
departamento de residencia y aplicar el factor de expansión
poblacional (`FACPOB07`) se hace un cruce persona a persona con el
Módulo 02, usando como llaves las variables `AÑO`, `CONGLOME`,
`VIVIENDA`, `HOGAR` y `CODPERSO`.

**¿Por qué esta combinación y no otra del mismo módulo?**

El bloque migratorio del Módulo 04 incluye también la pregunta P401F
("Hace 5 años, ¿vivía en este distrito?") y su complementaria P401G
(lugar donde vivía hace 5 años). Esa pareja capta sólo los movimientos
de los últimos 5 años, lo que genera una **ventana móvil distinta cada
año** y excluye a niños pequeños y a migrantes con más de 5 años en el
país. La pareja P401G1/P401G2, en cambio, captura un **atributo
permanente** de la persona (dónde nació), es comparable entre años y
cubre a migrantes de cualquier antigüedad.

#### Empalme con ENPOVE y ENAHOPV

Para los años 2018, 2022 y 2024 el INEI realizó encuestas dedicadas a
la población venezolana residente en el Perú, más precisas en los
departamentos cubiertos que la ENAHO regular:

| Año | Encuesta dedicada | Departamentos cubiertos |
|-----|--------------------|-------------------------|
| 2018 | ENPOVE 2018 | Arequipa, Callao, Cusco, La Libertad, Lima, Tumbes (6) |
| 2022 | ENPOVE 2022 | Áncash, Arequipa, Callao, Ica, La Libertad, Lambayeque, Lima, Piura, Tumbes (9) |
| 2024 | ENAHOPV 2024 | Áncash, Arequipa, Callao, Ica, La Libertad, Lambayeque, Lima, Piura, Tumbes (9) |

Regla de empalme: en los departamentos cubiertos por la encuesta
dedicada, el stock se toma íntegramente de esa encuesta (por diseño
todos sus residentes son migrantes); en los departamentos no
cubiertos, se mantiene el valor ENAHO calculado con la combinación
P401G1/P401G2.


#### Posible subrepresentación del stock real

Los valores reportados **tienden a quedar por debajo del stock real**
de migrantes, principalmente porque:

- La ENAHO encuesta únicamente **viviendas particulares con residentes
  habituales**. Los migrantes recientes que se alojan en hoteles,
  albergues, pensiones o sin vivienda estable quedan fuera del marco
  muestral, sobre todo en los años sin ENPOVE/ENAHOPV (2019-2021 y
  2023).
- La combinación P401G1/P401G2 es un **proxy matrilineal** del lugar
  de nacimiento; algunos migrantes cuya madre vivía en el Perú al
  momento del nacimiento (pero que nacieron en el extranjero) no son
  detectados.
- En los años con ENPOVE/ENAHOPV, los departamentos cubiertos se
  contabilizan con la **población venezolana exclusivamente**, dejando
  fuera a los extranjeros no venezolanos residentes en esos
  departamentos.

---

### 2.2 `housing_price` — Precio promedio de alquiler mensual por dormitorio

Mide el **precio promedio mensual, en soles corrientes, por habitación
de uso exclusivo para dormir**, en el departamento y año
correspondientes. No es el alquiler bruto de la vivienda completa sino
una medida normalizada por tamaño del inmueble.

La base de cálculo es la siguiente pregunta del Módulo 01 de la ENAHO
(Características de la Vivienda y del Hogar):

> **P106:** *Si usted alquilara esta vivienda, ¿cuánto cree que le
> pagarían de alquiler mensual (en S/.)?*

Esta pregunta se formula incluso a los hogares que no son inquilinos
(propietarios, ocupantes, etc.), lo que permite contar con una
valuación del mercado residencial amplio.

El cálculo sigue los siguientes pasos:

1. Se toma el alquiler estimado `P106` declarado por cada vivienda.
2. Se divide por el **número de habitaciones utilizadas exclusivamente
   para dormir**, según el propio Módulo 01 de la ENAHO. Esto permite
   comparar precios entre viviendas de tamaños distintos.
3. Se calcula el **promedio departamental** del precio por dormitorio,
   ponderado con el factor de expansión del hogar.

#### Restricciones muestrales aplicadas

Es importante notar que el precio **no se calcula sobre el universo
total de viviendas**. Se aplican dos filtros para mejorar la
comparabilidad y evitar sesgos por inmuebles no residenciales:

- **Sólo zonas urbanas.** Se excluyen las viviendas rurales porque el
  mercado de alquiler rural tiene dinámicas muy distintas y, en muchas
  regiones, es prácticamente inexistente o sin precios comparables.
- **Sólo casas independientes y departamentos.** Se excluyen viviendas
  improvisadas, cuartos en quinta, locales no construidos para habitar
  y otras categorías no estándar, que distorsionan la estimación del
  precio de mercado formal.

Por tanto, el valor refleja el precio del mercado urbano formal de
vivienda residencial, y no el costo promedio del total del parque
habitacional del departamento.

---

### 2.3 `housing_price_RMV2007` — Precio de vivienda deflactado en RMV de enero 2007

Precio de la vivienda expresado en términos de **Remuneraciones Mínimas
Vitales (RMV) de 2007**. Como la RMV de enero de
2007 era de **S/ 500**, la conversión consiste en dividir el precio
nominal de cada año entre 500. De esta forma, se tiene una especie de "precio relativo".
---

## 3. Controles

Variables agregadas a nivel departamental y anual, utilizadas como
controles demográficos, sociales y económicos en los análisis.

| Variable | Descripción |
|----------|-------------|
| **`rurality`** | Porcentaje de la población del departamento que reside en zonas rurales (clasificación oficial del INEI). Valores entre 0 y 100. |
| **`median_age`** | Edad mediana de los habitantes del departamento. La mitad de la población tiene una edad menor y la otra mitad, una edad mayor que este valor. Es menos sensible a los extremos que la edad media. |
| **`share_old_pop`** | Proporción (o porcentaje) de la población que es **adulto mayor**. La definición estándar del INEI considera adulto mayor a la persona de 60 años o más. |
| **`avg_HHsize`** | Tamaño promedio del hogar en el departamento: número promedio de miembros por hogar. |
| **`share_workingage_pop`** | Proporción de la **Población en Edad de Trabajar (PET)**. En la definición oficial del INEI, la PET comprende a las personas de 14 años a más. |
| **`annual_median_income`** | Ingreso mediano anual (en soles corrientes) de los hogares (o de los individuos, según el nivel del cálculo). La mediana se prefiere sobre el promedio porque es robusta frente a valores extremos. |
| **`population`** | Población total estimada del departamento. Proviene de las proyecciones oficiales del INEI basadas en Censos y ENAHO. |
| **`poverty_rate`** | Porcentaje de la población del departamento clasificado como **pobre monetario**, según la línea de pobreza oficial del INEI (construida a partir del gasto per cápita del hogar frente a una canasta básica mínima). |

---

## 4. Covariables Educativas

Proporciones de la población del departamento según el **nivel
educativo alcanzado**, siguiendo las 11 categorías estándar derivadas
del Módulo 03 de la ENAHO (Educación). Las 11 variables suman
aproximadamente 1 (o 100 %, según la escala) en cada departamento-año.

| Variable | Nivel educativo alcanzado |
|----------|----------------------------|
| **`d_educ_1`** | Sin nivel |
| **`d_educ_2`** | Inicial |
| **`d_educ_3`** | Primaria incompleta |
| **`d_educ_4`** | Primaria completa |
| **`d_educ_5`** | Secundaria incompleta |
| **`d_educ_6`** | Secundaria completa |
| **`d_educ_7`** | Superior no universitaria incompleta |
| **`d_educ_8`** | Superior no universitaria completa |
| **`d_educ_9`** | Superior universitaria incompleta |
| **`d_educ_10`** | Superior universitaria completa |
| **`d_educ_11`** | Maestría / doctorado |

### Interpretación

- Los niveles **1 a 6** corresponden al ciclo de educación básica
  regular (inicial, primaria y secundaria).
- Los niveles **7 y 8** corresponden a institutos técnicos o
  pedagógicos (educación superior no universitaria).
- Los niveles **9 y 10** corresponden a la educación universitaria de
  pregrado.
- El nivel **11** agrupa la educación de posgrado (maestría y
  doctorado), que en la ENAHO suele tener muy pocas observaciones por
  departamento — los coeficientes asociados deben leerse con cautela.

Una lectura rápida de la distribución ayuda a caracterizar el capital
humano del departamento: una región con `d_educ_3` y `d_educ_5` altos
concentra su población en ciclos educativos incompletos, mientras que
`d_educ_10` y `d_educ_11` elevados indican presencia de capital humano
altamente calificado.
