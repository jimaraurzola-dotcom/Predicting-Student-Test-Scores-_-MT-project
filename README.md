# Predicting Student Test Scores

Proyecto del curso de métodos estadísticos de Jimara Urzola y Jose Castro. Es un libro en `bookdown` que busca responder qué factores académicos, conductuales y de acceso a recursos influyen en el desempeño académico (`exam_score`) de los estudiantes, y en qué magnitud.

- Libro publicado: <https://jimaraurzola-dotcom.github.io/Predicting-Student-Test-Scores-_-MT-project/>
- Notas del curso en las que se basa el análisis: <https://cdeoroaguado.github.io/rbook_met_estad/>

## Datos

Conjunto sintético *Predicting Student Test Scores* (`Datos/train.csv`): 630.000 estudiantes, 4 variables numéricas (`age`, `study_hours`, `class_attendance`, `sleep_hours`), 7 categóricas (`gender`, `course`, `internet_access`, `sleep_quality`, `study_method`, `facility_rating`, `exam_difficulty`) y la variable respuesta `exam_score`, de 0 a 100.

## Contenido

1. Análisis exploratorio de datos (EDA)
2. Estadística paramétrica y no paramétrica: Lilliefors, Levene, U de Mann-Whitney, Kruskal-Wallis y post-hoc de Bonferroni
3. Tabla de contingencia: prueba $\chi^2$ de independencia entre aprobar o reprobar y las variables categóricas
4. Correlación lineal: coeficiente de Spearman
5. Conclusiones

## Resultado principal

`study_hours` es la variable más relacionada con la nota ($\rho = 0,77$). Le siguen el método de estudio, la calidad del sueño, la calidad de las instalaciones y la asistencia a clases. La edad, el género, el programa académico, la dificultad del examen y el acceso a internet no tienen relevancia práctica.

## Cómo renderizar

En la consola de RStudio, desde la carpeta del proyecto:

```r
bookdown::render_book()
```

El libro se genera en `docs/`. `Readme.Rmd` es el diario de avances del proyecto.
