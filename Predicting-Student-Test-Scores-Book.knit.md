---
title: "Predicting Student Test Scores"
author: "Jimara Urzola"
date: "2026-09-15"
site: bookdown::bookdown_site
output: bookdown::gitbook
documentclass: book
github-repo: jimaraurzola-dotcom/Predicting-Student-Test-Scores-_-MT-project
---

# Introducción

Para el desarrollo del análisis trabajaremos con el conjunto de datos Predicting Student Test Scores, un conjunto de datos sintético diseñado para problemas de regresión en el ámbito educativo. Este conjunto contiene información sobre características académicas, conductuales y de acceso a recursos de 630.000 estudiantes, con el objetivo de analizar y predecir el desempeño académico a partir de variables como las horas de estudio, la asistencia a clases, el método de estudio empleado y el acceso a recursos como internet.

A partir de este conjunto de datos, el presente proyecto busca responder la siguiente pregunta de investigación:

¿Cuáles son los factores académicos, conductuales y de acceso a recursos que influyen significativamente en el desempeño académico (exam_score) de los estudiantes, y en qué magnitud lo hacen?

### Contexto de los datos de student test scores

Las variables del conjunto de datos son:

* `age`: Edad del estudiante.

* `gender`: Género del estudiante (female, male, other).

* `study_hours`: Número de horas dedicadas al estudio.

* `study_method`: Método de estudio empleado (coaching, group study, mixed, online videos, self-study).

* `class_attendance`: Porcentaje de asistencia a clases.

* `sleep_hours`: Número de horas de sueño.

* `sleep_quality`: Calidad del sueño percibida (poor, average, good).

* `course`: Programa académico que cursa el estudiante.

* `internet_access`: Indica si el estudiante cuenta con acceso a internet (yes, no).

* `facility_rating`: Calificación de las instalaciones educativas (low, medium, high).

* `exam_difficulty`: Dificultad percibida del examen (easy, moderate, hard).

* `exam_score`: Calificación obtenida por el estudiante en el examen, en una escala de 0 a 100.


Iniciaremos con un análisis exploratorio de datos (EDA) del conjunto Predicting Student Test Scores, con el propósito de comprender la estructura de los datos, identificar patrones, detectar valores atípicos y analizar las relaciones entre las variables que pueden influir en el desempeño académico de los estudiantes.




<!--chapter:end:index.Rmd-->

<style>
body {
       background-color: #ffe4e1;
       font-family: Arial, sans-serif;
}

h1, h2 {
       color: f4f4f4;
}

a {
       color: f4f4f4;
       text-decoration: none;
}
a: hover {
       text-decoration: underline;
}
</style>

<style type="text/css">
body {
  background-color: #F8F4FF;
}
</style>

Carguemos las librerias y el conjunto de los datos


``` r
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ───────────────────────────────────────────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.2.1     ✔ readr     2.2.0
## ✔ forcats   1.0.1     ✔ stringr   1.6.0
## ✔ ggplot2   4.0.3     ✔ tibble    3.3.1
## ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
## ✔ purrr     1.2.2     
## ── Conflicts ─────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
library(Amelia)
```

```
## Loading required package: Rcpp
## ## 
## ## Amelia II: Multiple Imputation
## ## (Version 1.8.3, built: 2024-11-07)
## ## Copyright (C) 2005-2026 James Honaker, Gary King and Matthew Blackwell
## ## Refer to http://gking.harvard.edu/amelia/ for more information
## ##
```

``` r
library(moments)
library(patchwork)
library(GGally)
library(nortest)
library(rstatix)
```

```
## 
## Attaching package: 'rstatix'
## 
## The following object is masked from 'package:stats':
## 
##     filter
```


``` r
trainset <- read.csv('Datos/train.csv')
```


Verifiquemos que leímos bien los datos observando las primeras filas:


``` r
head(trainset, n=5)
```

```
##   id age gender  course study_hours class_attendance internet_access sleep_hours sleep_quality  study_method
## 1  0  21 female    b.sc        7.91             98.8              no         4.9       average online videos
## 2  1  18  other diploma        4.95             94.8             yes         4.7          poor    self-study
## 3  2  20 female    b.sc        4.68             92.6             yes         5.8          poor      coaching
## 4  3  19   male    b.sc        2.00             49.5             yes         8.3       average   group study
## 5  4  23   male     bca        7.65             86.9             yes         9.6          good    self-study
##   facility_rating exam_difficulty exam_score
## 1             low            easy       78.3
## 2          medium        moderate       46.7
## 3            high        moderate       99.0
## 4            high        moderate       63.9
## 5            high            easy      100.0
```

Con esto podemos confirmar que la base de datos fue cargada correctamente, como prueba de ello observando las primeras 5 filas de información.



``` r
colnames(trainset)
```

```
##  [1] "id"               "age"              "gender"           "course"           "study_hours"     
##  [6] "class_attendance" "internet_access"  "sleep_hours"      "sleep_quality"    "study_method"    
## [11] "facility_rating"  "exam_difficulty"  "exam_score"
```


``` r
str(trainset)
```

```
## 'data.frame':	630000 obs. of  13 variables:
##  $ id              : int  0 1 2 3 4 5 6 7 8 9 ...
##  $ age             : int  21 18 20 19 23 24 20 22 22 18 ...
##  $ gender          : chr  "female" "other" "female" "male" ...
##  $ course          : chr  "b.sc" "diploma" "b.sc" "b.sc" ...
##  $ study_hours     : num  7.91 4.95 4.68 2 7.65 5.04 4.28 4.19 1.06 3.44 ...
##  $ class_attendance: num  98.8 94.8 92.6 49.5 86.9 85.1 87 44.9 98.3 80.9 ...
##  $ internet_access : chr  "no" "yes" "yes" "yes" ...
##  $ sleep_hours     : num  4.9 4.7 5.8 8.3 9.6 9.4 9.1 8.8 5 6.2 ...
##  $ sleep_quality   : chr  "average" "poor" "poor" "average" ...
##  $ study_method    : chr  "online videos" "self-study" "coaching" "group study" ...
##  $ facility_rating : chr  "low" "medium" "high" "high" ...
##  $ exam_difficulty : chr  "easy" "moderate" "moderate" "moderate" ...
##  $ exam_score      : num  78.3 46.7 99 63.9 100 70.1 63.4 76.8 46.7 58.2 ...
```

Identifiquemos si existen valores `NA` por columna:


``` r
trainset %>%
  summarise(across(everything(), ~ sum(is.na(.))))
```

```
##   id age gender course study_hours class_attendance internet_access sleep_hours sleep_quality study_method
## 1  0   0      0      0           0                0               0           0             0            0
##   facility_rating exam_difficulty exam_score
## 1               0               0          0
```

Comprobamos con las variables cátegoricas.


``` r
trainset %>%
  select(where(is.character)) %>%
  summarise(across(everything(), ~ sum(is.na(.)), .names = "NA_{.col}"))
```

```
##   NA_gender NA_course NA_internet_access NA_sleep_quality NA_study_method NA_facility_rating NA_exam_difficulty
## 1         0         0                  0                0               0                  0                  0
```



``` r
missmap(trainset)
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-3-1.png" alt="" width="672" />



No existe ningun dato faltante en el dataset `train.csv`.

# Análisis exploratorio de datos (EDA)

## Análisis exploratorio de `exam_score` (Target)


``` r
   trainset %>% summarise(n = length(exam_score),
                 media = mean(exam_score),
                 sd = sd(exam_score),
                 mediana = median(exam_score),
                 RIC = IQR(exam_score),
                 Q1 = quantile(exam_score,0.25),
                 Q3 = quantile(exam_score,0.75),
                 minimo = min(exam_score),
                 maximo = max(exam_score),
                 asi = skewness(exam_score),
                 )
```

```
##        n    media       sd mediana  RIC   Q1   Q3 minimo maximo         asi
## 1 630000 62.50667 18.91688    62.6 27.5 48.8 76.3 19.599    100 -0.04827309
```

La variable `exam_score` fue analizada a partir de 630.000 estudiantes, donde cada fila representa un estudiante individual. El puntaje promedio es de 62,51 puntos, con una desviación estándar de 18,92 puntos, lo que refleja una dispersión moderada en el desempeño de los estudiantes.

El 50% de los estudiantes obtiene un puntaje menor o igual a 62,60 puntos, un valor prácticamente idéntico a la media, y el RIC de 27,5 puntos (entre el primer cuartil de 48,80 y el tercer cuartil de 76,30) muestra que el 50% central de los estudiantes se concentra en una franja relativamente estrecha. Esta cercanía entre la media y el 50% ya anticipa una distribución simétrica, confirmada por el coeficiente de asimetría de -0,05, casi nulo.

En los extremos, el estudiante con peor desempeño obtuvo 19,60 puntos, mientras que al menos uno alcanzó el máximo posible de 100. Esto es consistente con lo observado en el histograma, donde los puntajes se distribuyen de forma bastante uniforme alrededor del centro, sin un pico muy pronunciado ni colas extendidas.


* Histograma de densidad de la variable:


``` r
trainset %>%
  ggplot(aes(x = exam_score))+
  geom_histogram(aes(y = after_stat(density)),
                     binwidth = 5,
                      fill = "#c5b0ff",
                      color = "white",
                      alpha = 0.6)+
  geom_density(color = "darkblue", linewidth = 1.2) +
  coord_cartesian(xlim = c(0, 100), expand = FALSE) +
  labs(title = 'Distribución de la nota media del examen',
       x = 'Nota media del examen',
       y = 'Densidad')+
  theme_bw()
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/exscore histograma-1.png" alt="" width="672" />

Es posible apreciar que la moda se concentra entre 50 y 75 puntos de score.




* Boxplot


``` r
trainset %>%
  ggplot(aes(x = "", y= exam_score))+
  geom_boxplot(fill = "#c5b0ff", color = "darkblue",
               outlier.color = '#2f0be0')+
  stat_summary(fun = mean, geom = 'point', shape = 20,
               size = 3, color = 'black') +
  theme_bw()
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/exscore boxplot-1.png" alt="" width="672" />

El boxplot confirma lo visto en el histograma: la caja es simétrica, con la mediana (62,6) prácticamente centrada entre Q1 (48,8) y Q3 (76,3). Los bigotes no muestran valores atípicos.


## Análisis de las variables independientes (características)

* Inicialmente, analizaremos las variables númericas.


``` r
resumen_numericas <- trainset %>%
  select(where(is.numeric), -exam_score) %>%
  pivot_longer(
    cols = everything(),
    names_to = "variable",
    values_to = "valor"
  ) %>%
  group_by(variable) %>%
  summarise(
    n = sum(!is.na(valor)),
    media = round(mean(valor, na.rm = TRUE),3),
    ds = round(sd(valor, na.rm = TRUE),3),
    mediana = median(valor, na.rm = TRUE),
    minimo = min(valor, na.rm = TRUE),
    maximo = max(valor, na.rm = TRUE),
    Q1 = quantile(valor, 0.25, na.rm = TRUE),
    Q3 = quantile(valor, 0.75, na.rm = TRUE),
    IQR = IQR(valor, na.rm = TRUE),
    .groups = "drop"
  )

resumen_numericas
```

```
## # A tibble: 5 × 10
##   variable              n     media        ds  mediana minimo    maximo        Q1        Q3       IQR
##   <chr>             <int>     <dbl>     <dbl>    <dbl>  <dbl>     <dbl>     <dbl>     <dbl>     <dbl>
## 1 age              630000     20.5       2.26     21    17        24        19        23         4   
## 2 class_attendance 630000     72.0      17.4      72.6  40.6      99.4      57        87.2      30.2 
## 3 id               630000 315000.   181865.   315000.    0    629999    157500.   472499.   315000.  
## 4 sleep_hours      630000      7.07      1.74      7.1   4.1       9.9       5.6       8.6       3   
## 5 study_hours      630000      4.00      2.36      4     0.08      7.91      1.97      6.05      4.08
```


``` r
library(patchwork)

# Boxplot de age
p1 <- trainset %>%
  ggplot(aes(x = "", y = age)) +
  geom_boxplot(alpha = 0.7) +
  labs(title = "Edad de los estudiantes", y = "Edad", x = "") +
  theme_bw() +
  theme(plot.title = element_text(size = 10, face = "bold"))

# Boxplot de study_hours
p2 <- trainset %>%
  ggplot(aes(x = "", y = study_hours)) +
  geom_boxplot(alpha = 0.7) +
  labs(title = "Horas de estudio", y = "Horas", x = "") +
  theme_bw() +
  theme(plot.title = element_text(size = 10, face = "bold"))

# Boxplot de class_attendance
p3 <- trainset %>%
  ggplot(aes(x = "", y = class_attendance)) +
  geom_boxplot(alpha = 0.7) +
  labs(title = "Porcentaje de asistencia", y = "%", x = "") +
  theme_bw() +
  theme(plot.title = element_text(size = 10, face = "bold"))

# Boxplot de sleep_hours
p4 <- trainset %>%
  ggplot(aes(x = "", y = sleep_hours)) +
  geom_boxplot(alpha = 0.7) +
  labs(title = "Horas de sueño", y = "Horas", x = "") +
  theme_bw() +
  theme(plot.title = element_text(size = 10, face = "bold"))

# Cuadrícula 2x2
(p1 + p2) / (p3 + p4)
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/ind bxplot-1.png" alt="" width="672" />

Los boxplots muestran que las cuatro variables numéricas independientes tienen distribuciones bastante simétricas, sin valores atípicos notorios.

* `age`: la mayoría de los estudiantes tiene entre 19 y 23 años; el 50% tiene una edad menor o igual a 21 años.

* `study_hours`: las horas de estudio varían bastante entre estudiantes, desde casi 0 hasta cerca de 8 horas; el 50% dedica 4 horas o menos al estudio.

* `class_attendance`: la asistencia se concentra entre 57% y 87%; el 50% de los estudiantes tiene una asistencia menor o igual a 72,6%.

* `sleep_hours`: la mayoría de los estudiantes duerme entre 5,6 y 8,6 horas; el 50% duerme 7,1 horas o menos, un rango bastante saludable.

En general, ninguna de las cuatro variables muestra sesgos fuertes ni valores extremos que llamen la atención, lo cual es consistente con lo observado en las tablas resume



``` r
# Histograma de age
trainset %>%
  ggplot(aes(x = age)) +
  geom_histogram(fill = "#c5b0ff", alpha = 0.7, bins = 30, color = "white") +
  labs(title = "Edad de los estudiantes", x = "Edad", y = "Frecuencia") +
  theme_bw() +
  theme(plot.title = element_text(size = 10, face = "bold"))
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-4-1.png" alt="" width="672" />

La distribución muestra barras de altura muy similar entre los 17 y 24 años, confirmando que se trata de una variable discreta y prácticamente uniforme; No hay una edad que predomine claramente sobre las demás, lo que es consistente con lo observado en el resumen numérico (rango estrecho y baja dispersión).




``` r
# Histograma de study_hours
trainset %>%
  ggplot(aes(x = study_hours)) +
  geom_histogram(fill = "#c5b0ff", alpha = 0.7, bins = 50, color = "white") +
  labs(title = "Horas de estudio", x = "Horas", y = "Frecuencia") +
  theme_bw() +
  theme(plot.title = element_text(size = 10, face = "bold"))
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-5-1.png" alt="" width="672" />

la distribución es bastante uniforme a lo largo de todo el rango (0 a 8 horas), sin un pico claramente dominante, aunque se observan algunas barras puntuales con mayor frecuencia. Esto sugiere que los estudiantes se reparten de forma pareja entre quienes dedican pocas y muchas horas al estudio.



``` r
# Histograma de class_attendance
trainset %>%
  ggplot(aes(x = class_attendance)) +
  geom_histogram(fill = "#c5b0ff", alpha = 0.7, bins = 50, color = "white") +
  labs(title = "Porcentaje de asistencia", x = "%", y = "Frecuencia") +
  theme_bw() +
  theme(plot.title = element_text(size = 10, face = "bold"))
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-6-1.png" alt="" width="672" />

Se observa una distribución relativamente uniforme entre 40% y 90%, con un incremento notorio hacia el extremo superior (cerca del 100%), lo que indica que existe un grupo considerable de estudiantes con asistencia casi perfecta.



``` r
# Histograma de sleep_hours
trainset %>%
  ggplot(aes(x = sleep_hours)) +
  geom_histogram(fill = "#c5b0ff", alpha = 0.7, bins = 50, color = "white") +
  labs(title = "Horas de sueño", x = "Horas", y = "Frecuencia") +
  theme_bw() +
  theme(plot.title = element_text(size = 10, face = "bold"))
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-7-1.png" alt="" width="672" />

La distribución es bastante homogénea entre 4 y 10 horas, sin un patrón de concentración marcado, reflejando que los estudiantes duermen una cantidad de horas bastante variada, sin un comportamiento dominante.




* A continuación, trabajaremos en los gráficos de las variables categoricas.


``` r
tabla_study_method <- trainset %>%
  count(study_method, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 2),
    Variable = "study_method",
    Categoria = as.character(study_method)
  ) %>%
  select(Variable, Categoria, Frecuencia, Porcentaje)

tabla_study_method
```

```
##       Variable     Categoria Frecuencia Porcentaje
## 1 study_method      coaching     131697      20.90
## 2 study_method   group study     123009      19.53
## 3 study_method         mixed     123086      19.54
## 4 study_method online videos     121077      19.22
## 5 study_method    self-study     131131      20.81
```


``` r
tabla_study_method <- trainset %>%
  count(study_method, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 1),
    Etiqueta = paste0(Frecuencia, " (", Porcentaje, "%)")
  )

ggplot(tabla_study_method, aes(x = reorder(study_method, -Frecuencia), y = Frecuencia)) +
  geom_col(fill = "#c5b0ff", width = 0.4) +
  geom_text(aes(label = Etiqueta), vjust = -0.5, size = 3.5) +
  facet_wrap(~ "Distribución de la variable study_method") +
  scale_y_continuous(expand = expansion(mult = c(0, 0.15))) +
  labs(x = "Método de estudio", y = "Frecuencia (Porcentaje)") +
  theme_bw(base_size = 12) +
  theme(
    plot.title = element_blank(),
    strip.background = element_rect(fill = "gray80", color = NA),
    strip.text = element_text(face = "bold"),
    panel.grid.major.x = element_blank()
  )
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-8-1.png" alt="" width="672" />

Las cinco categorías de `study_method` están bastante equilibradas entre sí: coaching (20,90%) y self-study (20,81%) son las más frecuentes, seguidas de cerca por mixed (19,54%), group study (19,53%) y online videos (19,22%). A diferencia de otras variables del dataset, aquí no se observa ninguna categoría dominante ni ninguna subrepresentada, lo que sugiere una distribución prácticamente uniforme de los métodos de estudio entre los estudiantes.




``` r
tabla_gender <- trainset %>%
  count(gender, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 2),
    Variable = "gender",
    Categoria = as.character(gender)
  ) %>%
  select(Variable, Categoria, Frecuencia, Porcentaje)

tabla_gender
```

```
##   Variable Categoria Frecuencia Porcentaje
## 1   gender    female     208310      33.07
## 2   gender      male     210593      33.43
## 3   gender     other     211097      33.51
```


``` r
tabla_gender <- trainset %>%
  count(gender, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 1),
    Etiqueta = paste0(Frecuencia, " (", Porcentaje, "%)")
  )

ggplot(tabla_gender, aes(x = reorder(gender, -Frecuencia), y = Frecuencia)) +
  geom_col(fill = "#c5b0ff", width = 0.3) +
  geom_text(aes(label = Etiqueta), vjust = -0.5, size = 3) +
  facet_wrap(~ "Distribución de la variable gender") +
  scale_y_continuous(expand = expansion(mult = c(0, 0.15))) +
  labs(x = "Género", y = "Frecuencia (Porcentaje)") +
  theme_bw(base_size = 12) +
  theme(
    plot.title = element_blank(),
    strip.background = element_rect(fill = "gray80", color = NA),
    strip.text = element_text(face = "bold"),
    panel.grid.major.x = element_blank()
  )
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-10-1.png" alt="" width="672" />

La variable gender presenta una distribución muy pareja entre sus tres categorías: other (33,51%), male (33,43%) y female (33,07%), con diferencias de apenas 0,4 puntos porcentuales entre ellas. No se observa ningún desbalance relevante.



``` r
tabla_internet_access <- trainset %>%
  count(internet_access, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 2),
    Variable = "internet_access",
    Categoria = as.character(internet_access)
  ) %>%
  select(Variable, Categoria, Frecuencia, Porcentaje)

tabla_internet_access
```

```
##          Variable Categoria Frecuencia Porcentaje
## 1 internet_access        no      50577       8.03
## 2 internet_access       yes     579423      91.97
```


``` r
tabla_internet_access <- trainset %>%
  count(internet_access, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 1),
    Etiqueta = paste0(Frecuencia, " (", Porcentaje, "%)")
  )

ggplot(tabla_internet_access, aes(x = reorder(internet_access, -Frecuencia), y = Frecuencia)) +
  geom_col(fill = "#c5b0ff", width = 0.3) +
  geom_text(aes(label = Etiqueta), vjust = -0.5, size = 3.5) +
  facet_wrap(~ "Distribución de la variable internet_access") +
  scale_y_continuous(expand = expansion(mult = c(0, 0.6))) +
  labs(x = "Acceso a internet", y = "Frecuencia (Porcentaje)") +
  theme_bw(base_size = 12) +
  theme(
    plot.title = element_blank(),
    strip.background = element_rect(fill = "gray80", color = NA),
    strip.text = element_text(face = "bold"),
    panel.grid.major.x = element_blank()
  )
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-12-1.png" alt="" width="672" />

Aquí sí hay un desbalance marcado: el 91,97% de los estudiantes cuenta con acceso a internet (yes), mientras que solo el 8,03% no lo tiene (no). Esto podría influir en la robustez de futuras comparaciones entre ambos grupos, dado el tamaño tan reducido de quienes no cuentan con acceso.



``` r
tabla_course <- trainset %>%
  count(course, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 2),
    Variable = "course",
    Categoria = as.character(course)
  ) %>%
  select(Variable, Categoria, Frecuencia, Porcentaje)

tabla_course
```

```
##   Variable Categoria Frecuencia Porcentaje
## 1   course     b.com     110932      17.61
## 2   course      b.sc     111554      17.71
## 3   course    b.tech     131236      20.83
## 4   course        ba      61989       9.84
## 5   course       bba      75644      12.01
## 6   course       bca      88721      14.08
## 7   course   diploma      49924       7.92
```


``` r
tabla_course <- trainset %>%
  count(course, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 1),
    Etiqueta = paste0(Frecuencia, " (", Porcentaje, "%)")
  )

ggplot(tabla_course, aes(x = reorder(course, -Frecuencia), y = Frecuencia)) +
  geom_col(fill = "#c5b0ff", width = 0.45) +
  geom_text(aes(label = Etiqueta), vjust = -0.5, size = 2.8) +
  facet_wrap(~ "Distribución de la variable course") +
  scale_y_continuous(expand = expansion(mult = c(0, 0.15))) +
  labs(x = "Curso", y = "Frecuencia (Porcentaje)") +
  theme_bw(base_size = 12) +
  theme(
    plot.title = element_blank(),
    strip.background = element_rect(fill = "gray80", color = NA),
    strip.text = element_text(face = "bold"),
    panel.grid.major.x = element_blank()
  )
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-14-1.png" alt="" width="672" />

La categoría b.tech concentra la mayor proporción de estudiantes (20,83%), seguida por b.sc (17,71%) y b.com (17,61%). En conjunto, estas tres representan cerca del 56% del total. En el otro extremo, diploma es la categoría menos representada (7,92%), seguida por ba (9,84%). A diferencia de internet_access, aquí el desbalance es más moderado, con 7 categorías que oscilan entre el 8% y el 21%.


``` r
tabla_sleep_quality <- trainset %>%
  count(sleep_quality, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 2),
    Variable = "sleep_quality",
    Categoria = as.character(sleep_quality)
  ) %>%
  select(Variable, Categoria, Frecuencia, Porcentaje)

tabla_sleep_quality
```

```
##        Variable Categoria Frecuencia Porcentaje
## 1 sleep_quality   average     203236      32.26
## 2 sleep_quality      good     213089      33.82
## 3 sleep_quality      poor     213675      33.92
```


``` r
tabla_sleep_quality <- trainset %>%
  count(sleep_quality, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 1),
    Etiqueta = paste0(Frecuencia, " (", Porcentaje, "%)")
  )

ggplot(tabla_sleep_quality, aes(x = reorder(sleep_quality, -Frecuencia), y = Frecuencia)) +
  geom_col(fill = "#c5b0ff", width = 0.3) +
  geom_text(aes(label = Etiqueta), vjust = -0.5, size = 3.5) +
  facet_wrap(~ "Distribución de la variable sleep_quality") +
  scale_y_continuous(expand = expansion(mult = c(0, 0.15))) +
  labs(x = "Calidad del sueño", y = "Frecuencia (Porcentaje)") +
  theme_bw(base_size = 12) +
  theme(
    plot.title = element_blank(),
    strip.background = element_rect(fill = "gray80", color = NA),
    strip.text = element_text(face = "bold"),
    panel.grid.major.x = element_blank()
  )
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-16-1.png" alt="" width="672" />

Las tres categorías de sleep_quality están distribuidas de forma casi idéntica: poor (33,92%), good (33,82%) y average (32,26%), con una diferencia máxima de apenas 1,7 puntos porcentuales entre ellas. No se evidencia ningún patrón de concentración hacia una categoría en particular.




``` r
tabla_facility_rating <- trainset %>%
  count(facility_rating, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 2),
    Variable = "facility_rating",
    Categoria = as.character(facility_rating)
  ) %>%
  select(Variable, Categoria, Frecuencia, Porcentaje)

tabla_facility_rating
```

```
##          Variable Categoria Frecuencia Porcentaje
## 1 facility_rating      high     203540      32.31
## 2 facility_rating       low     212378      33.71
## 3 facility_rating    medium     214082      33.98
```


``` r
tabla_facility_rating <- trainset %>%
  count(facility_rating, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 1),
    Etiqueta = paste0(Frecuencia, " (", Porcentaje, "%)")
  )

ggplot(tabla_facility_rating, aes(x = reorder(facility_rating, -Frecuencia), y = Frecuencia)) +
  geom_col(fill = "#c5b0ff", width = 0.3) +
  geom_text(aes(label = Etiqueta), vjust = -0.5, size = 3.5) +
  facet_wrap(~ "Distribución de la variable facility_rating") +
  scale_y_continuous(expand = expansion(mult = c(0, 0.15))) +
  labs(x = "Calificación de instalaciones", y = "Frecuencia (Porcentaje)") +
  theme_bw(base_size = 12) +
  theme(
    plot.title = element_blank(),
    strip.background = element_rect(fill = "gray80", color = NA),
    strip.text = element_text(face = "bold"),
    panel.grid.major.x = element_blank()
  )
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-18-1.png" alt="" width="672" />

De forma similar a sleep_quality, las tres categorías de facility_rating se reparten de manera equilibrada: medium (33,98%), low (33,71%) y high (32,31%), sin ninguna diferencia relevante entre ellas.



``` r
tabla_exam_difficulty <- trainset %>%
  count(exam_difficulty, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 2),
    Variable = "exam_difficulty",
    Categoria = as.character(exam_difficulty)
  ) %>%
  select(Variable, Categoria, Frecuencia, Porcentaje)

tabla_exam_difficulty
```

```
##          Variable Categoria Frecuencia Porcentaje
## 1 exam_difficulty      easy     176540      28.02
## 2 exam_difficulty      hard      99478      15.79
## 3 exam_difficulty  moderate     353982      56.19
```


``` r
tabla_exam_difficulty <- trainset %>%
  count(exam_difficulty, name = "Frecuencia") %>%
  mutate(
    Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 1),
    Etiqueta = paste0(Frecuencia, " (", Porcentaje, "%)")
  )

ggplot(tabla_exam_difficulty, aes(x = reorder(exam_difficulty, -Frecuencia), y = Frecuencia)) +
  geom_col(fill = "#c5b0ff", width = 0.3) +
  geom_text(aes(label = Etiqueta), vjust = -0.5, size = 3.5) +
  facet_wrap(~ "Distribución de la variable exam_difficulty") +
  scale_y_continuous(expand = expansion(mult = c(0, 0.15))) +
  labs(x = "Dificultad del examen", y = "Frecuencia (Porcentaje)") +
  theme_bw(base_size = 12) +
  theme(
    plot.title = element_blank(),
    strip.background = element_rect(fill = "gray80", color = NA),
    strip.text = element_text(face = "bold"),
    panel.grid.major.x = element_blank()
  )
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-20-1.png" alt="" width="672" />

Esta es la variable categórica con mayor desbalance después de internet_access: la categoría moderate concentra más de la mitad de las observaciones (56,19%), mientras que easy representa el 28,02% y hard apenas el 15,79%. Esto sugiere que la mayoría de los exámenes fueron catalogados con una dificultad intermedia, y muy pocos como difíciles.


## Análisis exploratorio bivariado

A continuación, analizaremos la relación entre la variable objetivo `exam_score` y las variables numéricas independientes, con el fin de identificar posibles patrones, tendencias y asociaciones que puedan ser útiles para la construcción de modelos.



``` r
# Diagrama de dispersión: age vs exam_score
trainset %>%
  ggplot(aes(x = age, y = exam_score)) +
  geom_point(alpha = 0.4, color = "#c5b0ff") +
  geom_smooth(method = "lm", formula = y ~ x, se = FALSE, color = "darkblue") +
  labs(
    x = "Edad media",
    y = "Calificación media del examen"
  ) +
  theme_bw() +
  facet_grid(. ~ "Dispersión entre la edad media y la calificación de los exámenes")
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-21-1.png" alt="" width="672" />

El diagrama de dispersión muestra que la calificación del examen se distribuye de forma similar en todas las edades (17 a 24 años), con puntos que cubren prácticamente todo el rango de 20 a 100 puntos en cada grupo etario. La línea de tendencia es casi plana, lo que sugiere que no existe una relación lineal relevante entre la edad y el desempeño académico: los estudiantes más jóvenes y los mayores obtienen calificaciones igual de variadas.


``` r
# Diagrama de dispersión: study_hours vs exam_score
trainset %>%
  ggplot(aes(x = study_hours, y = exam_score)) +
  geom_point(alpha = 0.4, color = "#c5b0ff") +
  geom_smooth(method = "lm", formula = y ~ x, se = FALSE, color = "darkblue") +
  labs(
    x = "Horas de estudio media",
    y = "Calificación media del examen"
  ) +
  theme_bw() +
  facet_grid(. ~ "Dispersión entre las horas de estudio media y la calificación de los exámenes")
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-22-1.png" alt="" width="672" />

El diagrama de dispersión muestra una relación positiva clara entre las horas de estudio y la calificación del examen: a medida que aumentan las horas dedicadas al estudio, la nota tiende a subir, reflejado en la marcada pendiente ascendente de la línea de tendencia. A diferencia de `age`, aquí la nube de puntos se concentra de forma más definida alrededor de la línea, especialmente en los valores intermedios, lo que sugiere que las horas de estudio están fuertemente asociadas con el desempeño académico de los estudiantes.




``` r
# Diagrama de dispersión: class_attendance vs exam_score
trainset %>%
  ggplot(aes(x = class_attendance, y = exam_score)) +
  geom_point(alpha = 0.4, color = "#c5b0ff") +
  geom_smooth(method = "lm", formula = y ~ x, se = FALSE, color = "darkblue") +
  labs(
    x = "asistencia a clases media",
    y = "Calificación media del examen"
  ) +
  theme_bw() +
  facet_grid(. ~ "Dispersión entre la asistencia a clases media y la calificación de los exámenes")
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-23-1.png" alt="" width="672" />

El diagrama de dispersión evidencia una relación positiva entre el porcentaje de asistencia a clases y la calificación del examen: la línea de tendencia muestra una pendiente ascendente, indicando que los estudiantes con mayor asistencia tienden a obtener mejores resultados. Sin embargo, la nube de puntos se mantiene bastante dispersa a lo largo de todo el rango de asistencia, lo que sugiere que, aunque existe una tendencia positiva, la asistencia por sí sola no explica completamente las diferencias en el desempeño académico.




``` r
# Diagrama de dispersión: sleep_hours vs exam_score
trainset %>%
  ggplot(aes(x = sleep_hours, y = exam_score)) +
  geom_point(alpha = 0.4, color = "#c5b0ff") +
  geom_smooth(method = "lm", formula = y ~ x, se = FALSE, color = "darkblue") +
  labs(
    x = "horas de sueño media",
    y = "Calificación media del examen"
  ) +
  theme_bw() +
  facet_grid(. ~ "Dispersión horas de sueño media y la calificación de los exámenes")
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/unnamed-chunk-24-1.png" alt="" width="672" />

El diagrama de dispersión muestra una línea de tendencia con una leve pendiente positiva, sugiriendo una relación débil entre las horas de sueño y la calificación del examen. A diferencia de `study_hours`, aquí los puntos se distribuyen de manera mucho más uniforme y dispersa a lo largo de todo el rango de horas de sueño, sin un patrón visual tan marcado, lo que indica que esta variable parece tener una influencia limitada sobre el desempeño académico de los estudiantes.


# Análisis de estadística paramétrica y no paramétrica


* ¿El hecho de contar con acceso a internet influye en el desempeño académico de los estudiantes?

Para responder esto, empecemos con `internet_access` vs `exam_score`


``` r
# dataframe de acceso
Ac <- trainset %>% 
  filter(internet_access == "yes")

# dataframe de no acceso
Nac <- trainset %>% 
  filter(internet_access == "no")
```

Debido a que la muestra es mayor de 5.000 descartamos el uso de la Shapiro-Wilk.


``` r
trainset %>%
  group_by(internet_access) %>%
  summarise(n = n(),
            est_lt = lillie.test(exam_score)$statistic,
            p_lt = lillie.test(exam_score)$p.value)
```

```
## # A tibble: 2 × 4
##   internet_access      n est_lt      p_lt
##   <chr>            <int>  <dbl>     <dbl>
## 1 no               50577 0.0279 5.04e-102
## 2 yes             579423 0.0236 0
```

La prueba de Lilliefors muestra que el puntaje del examen `exam_score` no presenta un comportamiento normal en ninguno de los dos grupos analizados. En el caso de los estudiantes con acceso a internet $(D = 0.023561, p-valor < 0.001)$; mientras que para los estudiantes sin acceso a internet $(D = 0.027935, p-valor < 0.001)$.



``` r
trainset %>%
  wilcox_test(exam_score ~ internet_access, paired = FALSE)
```

```
## # A tibble: 1 × 7
##   .y.        group1 group2    n1     n2    statistic      p
## * <chr>      <chr>  <chr>  <int>  <int>        <dbl>  <dbl>
## 1 exam_score no     yes    50577 579423 14565777034. 0.0266
```

La prueba de Mann-Whitney-Wilcoxon para muestras independientes nos indica que, con una confianza del 95%, se concluye que existe una diferencia estadísticamente significativa en el puntaje del examen `exam_score` entre los estudiantes con y sin acceso a internet $(W=14565777034, p-valor=0.0266)$.


``` r
trainset %>%
  wilcox_effsize(exam_score ~ internet_access, paired = FALSE)
```

```
## # A tibble: 1 × 7
##   .y.        group1 group2 effsize    n1     n2 magnitude
## * <chr>      <chr>  <chr>    <dbl> <int>  <int> <ord>    
## 1 exam_score no     yes    0.00279 50577 579423 small
```

Con base en el resultado del tamaño del efecto $r$, se obtiene un valor estimado de 0.0028 (magnitud: pequeña), lo que indica un efecto prácticamente nulo. Esto indica que, aunque la diferencia es estadísticamente significativa, no es relevante desde el punto de vista práctico; El acceso a internet no parece tener una influencia considerable en el desempeño académico de los estudiantes.

En conjunto, estos resultados indican que, si bien existe una diferencia estadísticamente significativa en `exam_score` según el acceso a internet, su magnitud es prácticamente nula, por lo que esta variable no representa un factor relevante para explicar el desempeño académico de los estudiantes. En otras palabras, el acceso a internet no parece influir de manera importante en el desempeño académico.


* ¿Existen diferencias en el desempeño académico según el género del estudiante?

Para esto analicemos `gender` vs `exam_score`:


``` r
# OTHER
Other <- trainset %>% 
  filter(gender == "other")

# MALE
Male <- trainset %>% 
  filter(gender == "male")

# FEMALE
Female <- trainset %>%
  filter (gender == "female")
```


``` r
trainset %>%
  group_by(gender) %>%
  summarise(n = n(),
            est_lt = lillie.test(exam_score)$statistic,
            p_lt = lillie.test(exam_score)$p.value)
```

```
## # A tibble: 3 × 4
##   gender      n est_lt      p_lt
##   <chr>   <int>  <dbl>     <dbl>
## 1 female 208310 0.0238 2.66e-303
## 2 male   210593 0.0219 3.31e-260
## 3 other  211097 0.0259 0
```

La prueba de Lilliefors muestra que el puntaje del examen `exam_score` no presenta un comportamiento normal en ninguno de los tres grupos analizados según género. Para el grupo "other" $(D = 0.025855, p-valor < 0.001)$; para el grupo masculino $(D = 0.021919, p-valor < 0.001)$; y para el grupo femenino $(D = 0.023756, p-valor < 0.001)$.


``` r
kruskal.test(exam_score ~ gender, data = trainset)
```

```
## 
## 	Kruskal-Wallis rank sum test
## 
## data:  exam_score by gender
## Kruskal-Wallis chi-squared = 81.54, df = 2, p-value < 2.2e-16
```

La prueba de Kruskal-Wallis nos indica que, con una confianza del 95%, se concluye que existe una diferencia estadísticamente significativa en el puntaje del examen `exam_score` entre al menos uno de los grupos de género $(χ²₍₂₎ = 81.54, p-valor < 0.001)$.


``` r
trainset %>%
  kruskal_effsize(exam_score ~ gender)
```

```
## # A tibble: 1 × 5
##   .y.             n  effsize method  magnitude
## * <chr>       <int>    <dbl> <chr>   <ord>    
## 1 exam_score 630000 0.000126 eta2[H] small
```

Con base en el resultado del tamaño del efecto $(η² = 0.000126, magnitud: pequeña)$, se obtiene un valor prácticamente nulo. Esto indica que, aunque la diferencia entre los grupos de género es estadísticamente significativa, no es relevante desde el punto de vista práctico — el género explica una proporción mínima de la variabilidad en el puntaje del examen, por lo que no parece ser un factor determinante en el desempeño académico de los estudiantes.

En conjunto, estos resultados indican que, si bien existe una diferencia estadísticamente significativa en `exam_score` entre los grupos de género, su magnitud es prácticamente nula, por lo que el género no representa un factor relevante para explicar el desempeño académico de los estudiantes. En otras palabras, el género no genera diferencias importantes en el desempeño académico.


* ¿La calidad del sueño percibida por el estudiante se relaciona con su desempeño en el examen?

Analicemos `sleep_quality` vs `exam_score`:


``` r
Good <- trainset %>% filter(sleep_quality == "good")
Average <- trainset %>% filter(sleep_quality == "average")
Poor <- trainset %>% filter(sleep_quality == "poor")
```


``` r
trainset %>%
  group_by(sleep_quality) %>%
  summarise(n = n(),
            est_lt = lillie.test(exam_score)$statistic,
            p_lt = lillie.test(exam_score)$p.value)
```

```
## # A tibble: 3 × 4
##   sleep_quality      n est_lt      p_lt
##   <chr>          <int>  <dbl>     <dbl>
## 1 average       203236 0.0243 4.51e-309
## 2 good          213089 0.0381 0        
## 3 poor          213675 0.0212 3.19e-246
```

La prueba de Lilliefors muestra que el puntaje del examen `exam_score` no presenta un comportamiento normal en ninguno de los tres grupos analizados según calidad del sueño. Para el grupo "average" $(D = 0.024268, p-valor < 0.001)$; para el grupo "good" $(D = 0.038146, p-valor < 0.001)$; y para el grupo "poor" $(D = 0.021183, p-valor < 0.001)$.


``` r
kruskal.test(exam_score ~ sleep_quality, data = trainset)
```

```
## 
## 	Kruskal-Wallis rank sum test
## 
## data:  exam_score by sleep_quality
## Kruskal-Wallis chi-squared = 33168, df = 2, p-value < 2.2e-16
```

La prueba de Kruskal-Wallis nos indica que, con una confianza del 95%, se concluye que existe una diferencia estadísticamente significativa en el puntaje del examen `exam_score` entre al menos uno de los grupos de calidad del sueño $(χ²₍₂₎ = 33168, p-valor < 0.001)$.


``` r
trainset %>%
  kruskal_effsize(exam_score ~ sleep_quality)
```

```
## # A tibble: 1 × 5
##   .y.             n effsize method  magnitude
## * <chr>       <int>   <dbl> <chr>   <ord>    
## 1 exam_score 630000  0.0526 eta2[H] small
```

Con base en el resultado del tamaño del efecto $(η² = 0,0526, magnitud: pequeña$, se obtiene un valor considerablemente mayor al observado en `internet_access` (0,0028) y `gender` (0,000126). Esto indica que la calidad del sueño explica una proporción algo mayor de la variabilidad en el puntaje del examen en comparación con las otras variables categóricas analizadas hasta ahora.

En conjunto, estos resultados indican que existe una diferencia estadísticamente significativa en `exam_score` entre los grupos de calidad del sueño, con un tamaño del efecto mayor al de las variables anteriores, aunque aún clasificado como pequeño. En otras palabras, la calidad del sueño sí se relaciona con el desempeño académico, aunque de forma modesta.


* ¿La calificación que los estudiantes dan a las instalaciones educativas está relacionada con su desempeño académico? 

Es necesario analizar `facility_rating` vs `exam_score`:


``` r
trainset %>%
  group_by(facility_rating) %>%
  summarise(n = n(),
            est_lt = lillie.test(exam_score)$statistic,
            p_lt = lillie.test(exam_score)$p.value)
```

```
## # A tibble: 3 × 4
##   facility_rating      n est_lt      p_lt
##   <chr>            <int>  <dbl>     <dbl>
## 1 high            203540 0.0362 0        
## 2 low             212378 0.0207 1.67e-232
## 3 medium          214082 0.0236 1.49e-308
```

La prueba de Lilliefors muestra que el puntaje del examen `exam_score` no presenta un comportamiento normal en ninguna de las tres categorías de calificación de instalaciones. Para `high` $(D = 0.0362, p-valor < 0.001)$; para `low` $(D = 0.0207, p-valor < 0.001)$; y para `medium` $(D = 0.0236, p-valor < 0.001)$.


``` r
kruskal.test(exam_score ~ facility_rating, data = trainset)
```

```
## 
## 	Kruskal-Wallis rank sum test
## 
## data:  exam_score by facility_rating
## Kruskal-Wallis chi-squared = 20684, df = 2, p-value < 2.2e-16
```

La prueba de Kruskal-Wallis nos indica que, con una confianza del 95%, se concluye que existe una diferencia estadísticamente significativa en el puntaje del examen `exam_score` entre al menos uno de los grupos de calificación de instalaciones $(χ²₍₂₎ = 20684, p-valor < 0.001)$.


``` r
trainset %>%
  kruskal_effsize(exam_score ~ facility_rating)
```

```
## # A tibble: 1 × 5
##   .y.             n effsize method  magnitude
## * <chr>       <int>   <dbl> <chr>   <ord>    
## 1 exam_score 630000  0.0328 eta2[H] small
```

Con base en el resultado del tamaño del efecto $(η² = 0,0328, magnitud: pequeña)$, se obtiene un valor intermedio entre lo observado en `study_method` (que veremos a continuación) y variables como `gender` o `internet_access`. Esto sugiere que la calidad de las instalaciones tiene alguna influencia sobre el desempeño académico, aunque, al igual que las demás variables categóricas, su efecto práctico sigue siendo pequeño.

En conjunto, estos resultados indican que existe una diferencia estadísticamente significativa en `exam_score` según `facility_rating`, con un tamaño del efecto pequeño, similar en orden de magnitud al de `sleep_quality`. En otras palabras, la calificación de las instalaciones sí está relacionada con el desempeño académico, aunque de forma limitada.



* ¿La dificultad percibida del examen afecta la calificación obtenida por los estudiantes?

Para responder esta pregunta debemos analizar `exam_difficulty` vs `exam_score`:


``` r
trainset %>%
  group_by(exam_difficulty) %>%
  summarise(n = n(),
            est_lt = lillie.test(exam_score)$statistic,
            p_lt = lillie.test(exam_score)$p.value)
```

```
## # A tibble: 3 × 4
##   exam_difficulty      n est_lt      p_lt
##   <chr>            <int>  <dbl>     <dbl>
## 1 easy            176540 0.0217 8.05e-214
## 2 hard             99478 0.0255 7.47e-168
## 3 moderate        353982 0.0243 0
```

La prueba de Lilliefors muestra que el puntaje del examen `exam_score` no presenta un comportamiento normal en ninguna de las tres categorías de dificultad del examen. Para `easy` $(D = 0.0217, p-valor < 0.001)$; para `hard` $(D = 0.0255, p-valor < 0.001)$; y para `moderate` $(D = 0.0243, p-valor < 0.001)$.


``` r
kruskal.test(exam_score ~ exam_difficulty, data = trainset)
```

```
## 
## 	Kruskal-Wallis rank sum test
## 
## data:  exam_score by exam_difficulty
## Kruskal-Wallis chi-squared = 53.954, df = 2, p-value = 1.924e-12
```

La prueba de Kruskal-Wallis nos indica que, con una confianza del 95%, se concluye que existe una diferencia estadísticamente significativa en el puntaje del examen `exam_score` entre al menos uno de los grupos de dificultad del examen $(χ²₍₂₎ = 53.954, p-valor < 0.001)$.


``` r
trainset %>%
  kruskal_effsize(exam_score ~ exam_difficulty)
```

```
## # A tibble: 1 × 5
##   .y.             n   effsize method  magnitude
## * <chr>       <int>     <dbl> <chr>   <ord>    
## 1 exam_score 630000 0.0000825 eta2[H] small
```

Con base en el resultado del tamaño del efecto $(η² = 0,0000825, magnitud: pequeña)$, se obtiene el valor más bajo de todas las variables categóricas analizadas hasta ahora, incluso menor que `gender` (0,000126). Esto indica que la dificultad percibida del examen prácticamente no explica variabilidad alguna en `exam_score`.

En conjunto, estos resultados indican que, si bien existe una diferencia estadísticamente significativa en `exam_score` según `exam_difficulty`, su magnitud es prácticamente nula, por lo que esta variable no representa un factor relevante para explicar el desempeño académico de los estudiantes. En otras palabras, la dificultad percibida del examen no afecta de manera importante la calificación obtenida.


* ¿El método de estudio empleado influye en el puntaje obtenido en el examen?

Analicemos `study_method` vs `exam_score`:


``` r
trainset %>%
  group_by(study_method) %>%
  summarise(n = n(),
            est_lt = lillie.test(exam_score)$statistic,
            p_lt = lillie.test(exam_score)$p.value)
```

```
## # A tibble: 5 × 4
##   study_method       n est_lt      p_lt
##   <chr>          <int>  <dbl>     <dbl>
## 1 coaching      131697 0.0478 0        
## 2 group study   123009 0.0199 1.75e-124
## 3 mixed         123086 0.0317 9.88e-324
## 4 online videos 121077 0.0238 1.05e-176
## 5 self-study    131131 0.0258 4.69e-225
```

La prueba de Lilliefors muestra que el puntaje del examen `exam_score` no presenta un comportamiento normal en ninguna de las cinco categorías de método de estudio. Para `coaching` $(D = 0.0478, p-valor < 0.001)$; para `group study` $(D = 0.0199, p-valor < 0.001)$; para `mixed` $(D = 0.0317, p-valor < 0.001)$; para `online videos` $(D = 0.0238, p-valor < 0.001)$; y para `self-study` $(D = 0.0258, p-valor < 0.001)$.


``` r
kruskal.test(exam_score ~ study_method, data = trainset)
```

```
## 
## 	Kruskal-Wallis rank sum test
## 
## data:  exam_score by study_method
## Kruskal-Wallis chi-squared = 29548, df = 4, p-value < 2.2e-16
```

La prueba de Kruskal-Wallis nos indica que, con una confianza del 95%, se concluye que existe una diferencia estadísticamente significativa en el puntaje del examen `exam_score` entre al menos uno de los grupos de método de estudio $(χ²₍₄₎ = 29548, p-valor < 0.001)$.


``` r
trainset %>%
  kruskal_effsize(exam_score ~ study_method)
```

```
## # A tibble: 1 × 5
##   .y.             n effsize method  magnitude
## * <chr>       <int>   <dbl> <chr>   <ord>    
## 1 exam_score 630000  0.0469 eta2[H] small
```

Con base en el resultado del tamaño del efecto $(η² = 0,0469, magnitud: pequeña)$, se obtiene el segundo valor más alto entre todas las variables categóricas analizadas, muy cerca del observado en `sleep_quality` (0,0526) y por encima de `facility_rating` (0,0328). Esto sugiere que el método de estudio, junto con la calidad del sueño, es de las variables categóricas con mayor relación (aunque todavía modesta) con el desempeño académico.

En conjunto, estos resultados indican que existe una diferencia estadísticamente significativa en exam_score según study_method, con un tamaño del efecto pequeño pero comparativamente de los más altos entre las variables categóricas del dataset. En otras palabras, el método de estudio sí influye en el desempeño académico, siendo una de las variables más relevantes encontradas.


* ¿El programa académico que cursa el estudiante se relaciona con su desempeño en el examen?

Finalicemos analizando `course` vs `exam_score`:


``` r
trainset %>%
  group_by(course) %>%
  summarise(n = n(),
            est_lt = lillie.test(exam_score)$statistic,
            p_lt = lillie.test(exam_score)$p.value)
```

```
## # A tibble: 7 × 4
##   course       n est_lt      p_lt
##   <chr>    <int>  <dbl>     <dbl>
## 1 b.com   110932 0.0226 5.06e-146
## 2 b.sc    111554 0.0254 1.91e-185
## 3 b.tech  131236 0.0254 1.20e-218
## 4 ba       61989 0.0278 3.70e-124
## 5 bba      75644 0.0275 4.20e-148
## 6 bca      88721 0.0240 1.25e-131
## 7 diploma  49924 0.0245 1.11e- 76
```

La prueba de Lilliefors muestra que el puntaje del examen `exam_score` no presenta un comportamiento normal en ninguna de las siete categorías de programa académico (en todos los casos $p-valor < 0.001$).


``` r
kruskal.test(exam_score ~ course, data = trainset)
```

```
## 
## 	Kruskal-Wallis rank sum test
## 
## data:  exam_score by course
## Kruskal-Wallis chi-squared = 199.28, df = 6, p-value < 2.2e-16
```

La prueba de Kruskal-Wallis nos indica que, con una confianza del 95%, se concluye que existe una diferencia estadísticamente significativa en el puntaje del examen `exam_score` entre al menos uno de los programas académicos $(χ²₍₆₎ = 199.28, p-valor < 0.001)$.


``` r
trainset %>%
  kruskal_effsize(exam_score ~ course)
```

```
## # A tibble: 1 × 5
##   .y.             n  effsize method  magnitude
## * <chr>       <int>    <dbl> <chr>   <ord>    
## 1 exam_score 630000 0.000307 eta2[H] small
```

Con base en el resultado del tamaño del efecto $(η² = 0,000307, magnitud: pequeña)$, se obtiene un valor prácticamente nulo, similar al de `gender` y muy por debajo del de `sleep_quality` o `study_method`. Esto indica que el programa académico que cursa el estudiante no explica una proporción relevante de la variabilidad en `exam_score`.

En conjunto, estos resultados indican que, si bien existe una diferencia estadísticamente significativa en `exam_score` según `course`, su magnitud es prácticamente nula, por lo que esta variable no representa un factor relevante para explicar el desempeño académico de los estudiantes. En otras palabras, el programa académico no se relaciona de manera importante con el desempeño de los estudiantes.

Con las siete variables categóricas ya analizadas, el panorama es bastante consistente: en todos los casos el p-valor resultó estadísticamente significativo, pero esto es esperable dado el tamaño de muestra (630.000 observaciones), donde hasta diferencias mínimas terminan "significativas". Lo que de verdad permite distinguir entre variables es el tamaño del efecto, y ahí el orden queda así: `sleep_quality` (η² = 0,0526) por encima de `study_method` (η² = 0,0469) y `facility_rating` (η² = 0,0328); bastante más atrás quedan `course` (η² = 0,000307), `gender` (η² = 0,000126), `exam_difficulty` (η² = 0,0000825) e `internet_access` (r = 0,0028), todas prácticamente nulas. Ni siquiera las tres primeras superan la magnitud "pequeña", así que ninguna variable categórica por sí sola explica gran cosa del desempeño académico. Esto confirma lo que ya había anotado en el diario del proyecto: con muestras tan grandes, el p-valor solo no sirve para decidir si algo es relevante, hay que mirar siempre el tamaño del efecto.

# Tabla de contingencia

Hasta ahora hemos comparado `exam_score` (numérica) entre los grupos de cada variable categórica. Una forma complementaria de explorar relaciones es a través de tablas de contingencia, que permiten estudiar la asociación entre dos variables categóricas usando la prueba χ² de independencia.

Para conectar este análisis con la pregunta de investigación del proyecto, creamos una versión categórica de la variable respuesta: `resultado`, que clasifica a cada estudiante como `aprobado` (`exam_score >= 60`) o `reprobado` (`exam_score < 60`).


``` r
trainset <- trainset %>%
  mutate(resultado = ifelse(exam_score >= 60, "aprobado", "reprobado"))

trainset %>%
  count(resultado, name = "Frecuencia") %>%
  mutate(Porcentaje = round(Frecuencia / sum(Frecuencia) * 100, 2))
```

```
##   resultado Frecuencia Porcentaje
## 1  aprobado     347803      55.21
## 2 reprobado     282197      44.79
```

El 55,21% de los estudiantes aprueba (`exam_score >= 60`) y el 44,79% reprueba, una distribución razonablemente balanceada para el análisis.

A continuación, evaluamos si `resultado` está asociado con las dos variables categóricas que mostraron el mayor tamaño del efecto en la sección anterior: `sleep_quality` y `study_method`.

* `resultado` vs `sleep_quality`


``` r
tabla_resultado_sq <- table(trainset$resultado, trainset$sleep_quality)
tabla_resultado_sq
```

```
##            
##             average   good   poor
##   aprobado   111912 140482  95409
##   reprobado   91324  72607 118266
```


``` r
chisq_sq <- chisq.test(tabla_resultado_sq)
chisq_sq
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  tabla_resultado_sq
## X-squared = 19531, df = 2, p-value < 2.2e-16
```

Antes de interpretar la prueba, verificamos el supuesto de χ²: todas las frecuencias esperadas deben ser mayores o iguales a 5.


``` r
min(chisq_sq$expected)
```

```
## [1] 91035.86
```

La frecuencia esperada mínima es de 91.035,86, muy por encima de 5, por lo que se cumple el supuesto y la prueba χ² es válida.

Con una confianza del 95%, la prueba χ² de independencia $(χ²₍₂₎ = 19531, p-valor < 0.001)$ nos indica que `resultado` y `sleep_quality` no son independientes; existe asociación estadísticamente significativa entre la calidad del sueño y el hecho de aprobar o reprobar el examen.


``` r
k_sq <- min(nrow(tabla_resultado_sq) - 1, ncol(tabla_resultado_sq) - 1)
v_sq <- sqrt(chisq_sq$statistic / (sum(tabla_resultado_sq) * k_sq))
v_sq
```

```
## X-squared 
## 0.1760708
```

La V de Cramer obtenida es de 0,1761, lo que indica una asociación débil entre ambas variables. Esto es consistente con el tamaño del efecto pequeño (η² = 0,0526) encontrado con Kruskal-Wallis en la sección anterior: la calidad del sueño está relacionada con el desempeño académico, pero de forma modesta.

* `resultado` vs `study_method`


``` r
tabla_resultado_sm <- table(trainset$resultado, trainset$study_method)
tabla_resultado_sm
```

```
##            
##             coaching group study mixed online videos self-study
##   aprobado     88790       63351 76249         59758      59655
##   reprobado    42907       59658 46837         61319      71476
```


``` r
chisq_sm <- chisq.test(tabla_resultado_sm)
chisq_sm
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  tabla_resultado_sm
## X-squared = 17569, df = 4, p-value < 2.2e-16
```


``` r
min(chisq_sm$expected)
```

```
## [1] 54234.23
```

La frecuencia esperada mínima es de 54.234,23, también muy por encima de 5, por lo que se cumple el supuesto de la prueba χ².

Con una confianza del 95%, la prueba χ² de independencia $(χ²₍₄₎ = 17569, p-valor < 0.001)$ nos indica que `resultado` y `study_method` tampoco son independientes.


``` r
k_sm <- min(nrow(tabla_resultado_sm) - 1, ncol(tabla_resultado_sm) - 1)
v_sm <- sqrt(chisq_sm$statistic / (sum(tabla_resultado_sm) * k_sm))
v_sm
```

```
## X-squared 
## 0.1669942
```

La V de Cramer para `study_method` es de 0,1670, prácticamente igual a la obtenida con `sleep_quality` (0,1761), lo que vuelve a confirmar una asociación débil.

En conjunto, ambas pruebas χ² resultan estadísticamente significativas, pero al igual que con Kruskal-Wallis, el tamaño del efecto (V de Cramer ≈ 0,17 en los dos casos) confirma que la asociación entre estas variables categóricas y el resultado del examen (`resultado`) es real pero débil. Ninguna de las dos alcanza sola para predecir con fuerza si un estudiante va a aprobar o no.

# Correlación

En el EDA ya habíamos observado, a través de diagramas de dispersión, que `study_hours` y `class_attendance` parecían tener una relación positiva con `exam_score`, mientras que `age` y `sleep_hours` no mostraban un patrón claro. En esta sección cuantificamos esas relaciones con el coeficiente de correlación.

Como ya establecimos que `exam_score` no sigue una distribución normal (Lilliefors, sección anterior), primero verificamos la normalidad de las cuatro variables numéricas independientes para decidir entre correlación de Pearson (paramétrica) o de Spearman (no paramétrica). Al igual que antes, descartamos Shapiro-Wilk por el tamaño de la muestra (630.000) y usamos Lilliefors.


``` r
c(age = lillie.test(trainset$age)$statistic,
  study_hours = lillie.test(trainset$study_hours)$statistic,
  class_attendance = lillie.test(trainset$class_attendance)$statistic,
  sleep_hours = lillie.test(trainset$sleep_hours)$statistic)
```

```
##              age.D      study_hours.D class_attendance.D      sleep_hours.D 
##         0.11515884         0.06276541         0.06111424         0.07380016
```

``` r
c(age = lillie.test(trainset$age)$p.value,
  study_hours = lillie.test(trainset$study_hours)$p.value,
  class_attendance = lillie.test(trainset$class_attendance)$p.value,
  sleep_hours = lillie.test(trainset$sleep_hours)$p.value)
```

```
##              age      study_hours class_attendance      sleep_hours 
##                0                0                0                0
```

Ninguna de las cuatro variables numéricas sigue una distribución normal (`age`: D = 0,1152; `study_hours`: D = 0,0628; `class_attendance`: D = 0,0611; `sleep_hours`: D = 0,0738; en los cuatro casos p-valor < 0,001). Por lo tanto, usamos el coeficiente de correlación de Spearman en lugar de Pearson, consistente con el enfoque no paramétrico usado en toda la sección anterior.

* Matriz de correlación de Spearman entre las variables numéricas:


``` r
numericas <- trainset %>%
  select(age, study_hours, class_attendance, sleep_hours, exam_score)

round(cor(numericas, method = "spearman"), 3)
```

```
##                    age study_hours class_attendance sleep_hours exam_score
## age              1.000       0.008            0.006       0.006      0.007
## study_hours      0.008       1.000            0.087       0.043      0.770
## class_attendance 0.006       0.087            1.000       0.029      0.352
## sleep_hours      0.006       0.043            0.029       1.000      0.160
## exam_score       0.007       0.770            0.352       0.160      1.000
```


``` r
ggcorr(numericas, method = c("pairwise", "spearman"),
       label = TRUE, label_round = 2,
       low = "#f8f4ff", mid = "#c5b0ff", high = "#2f0be0",
       hjust = 0.8, size = 4) +
  labs(title = "Matriz de correlación de Spearman")
```

<img src="Predicting-Student-Test-Scores-Book_files/figure-html/correlacion heatmap-1.png" alt="" width="672" />

La matriz confirma lo observado visualmente en el EDA: `study_hours` es, por lejos, la variable más correlacionada con `exam_score`; le sigue `class_attendance` con una correlación moderada-débil; `sleep_hours` y `age` muestran correlaciones muy débiles, tanto con `exam_score` como entre ellas.

* `study_hours` vs `exam_score`


``` r
cor.test(trainset$study_hours, trainset$exam_score, method = "spearman", exact = FALSE)
```

```
## 
## 	Spearman's rank correlation rho
## 
## data:  trainset$study_hours and trainset$exam_score
## S = 9.5952e+15, p-value < 2.2e-16
## alternative hypothesis: true rho is not equal to 0
## sample estimates:
##       rho 
## 0.7697575
```

Se obtiene un coeficiente $\rho = 0.7698$ $(S = 9.5952e+15, p-valor < 0.001)$. Con una confianza del 95%, existe evidencia de una correlación positiva muy fuerte y estadísticamente significativa entre las horas de estudio y la calificación del examen: a más horas de estudio, mayor es la nota. Este es, por lejos, el predictor numérico más fuerte del dataset.

* `class_attendance` vs `exam_score`


``` r
cor.test(trainset$class_attendance, trainset$exam_score, method = "spearman", exact = FALSE)
```

```
## 
## 	Spearman's rank correlation rho
## 
## data:  trainset$class_attendance and trainset$exam_score
## S = 2.7025e+16, p-value < 2.2e-16
## alternative hypothesis: true rho is not equal to 0
## sample estimates:
##      rho 
## 0.351515
```

Se obtiene un coeficiente $\rho = 0.3515$ $(S = 2.7025e+16, p-valor < 0.001)$, lo que corresponde a una correlación positiva débil-moderada: la asistencia a clases está relacionada con el desempeño académico, pero de forma bastante más modesta que las horas de estudio.

* `sleep_hours` vs `exam_score`


``` r
cor.test(trainset$sleep_hours, trainset$exam_score, method = "spearman", exact = FALSE)
```

```
## 
## 	Spearman's rank correlation rho
## 
## data:  trainset$sleep_hours and trainset$exam_score
## S = 3.5005e+16, p-value < 2.2e-16
## alternative hypothesis: true rho is not equal to 0
## sample estimates:
##       rho 
## 0.1600479
```

Se obtiene un coeficiente $\rho = 0.1600$ $(S = 3.5005e+16, p-valor < 0.001)$. Aunque estadísticamente significativa, la correlación es muy débil, casi nula en la práctica: dormir más o menos horas prácticamente no se traduce en diferencias en el puntaje del examen.

* `age` vs `exam_score`


``` r
cor.test(trainset$age, trainset$exam_score, method = "spearman", exact = FALSE)
```

```
## 
## 	Spearman's rank correlation rho
## 
## data:  trainset$age and trainset$exam_score
## S = 4.1376e+16, p-value = 1.303e-08
## alternative hypothesis: true rho is not equal to 0
## sample estimates:
##         rho 
## 0.007163204
```

Se obtiene un coeficiente $\rho = 0.0072$ $(S = 4.1376e+16, p-valor = 1.303e-08)$. A pesar de que el tamaño de muestra hace que hasta esta correlación tan pequeña resulte "significativa", el coeficiente es prácticamente cero: la edad del estudiante no tiene ninguna relación relevante con su desempeño en el examen, confirmando lo que ya se veía en el diagrama de dispersión del EDA.

En conjunto, de las cuatro variables numéricas independientes, `study_hours` es por mucho la que mejor se relaciona con `exam_score` (ρ = 0,77, correlación muy fuerte), seguida bastante más atrás por `class_attendance` (ρ = 0,35, débil-moderada). `sleep_hours` (ρ = 0,16) y `age` (ρ ≈ 0) prácticamente no aportan nada. Y comparando con las secciones anteriores, ninguna variable categórica (ni siquiera `sleep_quality` o `study_method`) se acerca a la fuerza de esta relación. Todo apunta a que `study_hours` va a ser el predictor más importante cuando lleguemos a la regresión lineal.


<!--chapter:end:Predicting-student-test-scores.Rmd-->

