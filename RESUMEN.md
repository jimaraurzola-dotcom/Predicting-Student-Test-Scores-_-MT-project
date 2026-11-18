# Resumen del proyecto

**Predicción de calificaciones de estudiantes**
Jimara Urzola y Jose Castro. Curso de métodos estadísticos.

Libro publicado: <https://jimaraurzola-dotcom.github.io/Predicting-Student-Test-Scores-_-MT-project/>

## Pregunta de investigación

¿Cuáles son los factores académicos, conductuales y de acceso a recursos que influyen significativamente en el desempeño académico (`exam_score`) de los estudiantes, y en qué magnitud lo hacen?

## Los datos

Conjunto sintético *Predicting Student Test Scores*: 630.000 estudiantes, sin datos faltantes.

| Tipo | Variables |
|---|---|
| Respuesta | `exam_score` (de 19,6 a 100; media 62,51; desviación estándar 18,92) |
| Numéricas | `study_hours`, `class_attendance`, `sleep_hours`, `age` |
| Categóricas | `gender`, `course`, `internet_access`, `sleep_quality`, `study_method`, `facility_rating`, `exam_difficulty` |

## Qué hicimos, capítulo por capítulo

| Capítulo | Qué se hizo | Resultado |
|---|---|---|
| EDA | Tablas resumen, histogramas, boxplots y diagramas de dispersión | `exam_score` es simétrica y sin atípicos. A simple vista pesan `study_hours`, `study_method`, `sleep_quality` y `facility_rating` |
| Paramétrica y no paramétrica | Lilliefors por grupo, Levene, U de Mann-Whitney, Kruskal-Wallis y post-hoc de Bonferroni | Ningún grupo es normal, así que se usan pruebas no paramétricas. Todas salen significativas, pero solo tres variables muestran diferencias grandes |
| Tabla de contingencia | $\chi^2$ de independencia entre aprobar (nota de 60 o más) y `sleep_quality` y `study_method`, paso a paso y con `chisq.test` | Aprobar no es independiente de ninguna de las dos |
| Correlación lineal | Lilliefors y coeficiente de Spearman | `study_hours` 0,77 (muy fuerte), `class_attendance` 0,35 (débil), `sleep_hours` 0,16 y `age` 0,01 (sin correlación) |
| Regresión lineal múltiple | VIF, variables dummy, `stepAIC`, coeficientes con matrices, ANOVA, pruebas $t$, intervalos, residuales y predicción | El modelo explica el 77,89% de la variabilidad de la nota |

## La idea que atraviesa todo el proyecto

Con 630.000 observaciones casi cualquier prueba sale significativa, incluso cuando la diferencia es de décimas de punto. Por eso la importancia de cada variable la juzgamos por la **magnitud** (diferencia entre medianas, coeficiente de correlación, tamaño del coeficiente) y no solo por el p-valor.

## Resultados por variable categórica

| Variable | Mediana más baja | Mediana más alta | Diferencia |
|---|---|---|---|
| `study_method` | 57,3 (self-study) | 69,3 (coaching) | 12 puntos |
| `sleep_quality` | 57,2 (poor) | 68,2 (good) | 11 puntos |
| `facility_rating` | 58,3 (low) | 67,3 (high) | 9 puntos |
| `gender`, `internet_access`, `exam_difficulty`, `course` | | | menos de 2 puntos |

Porcentaje de aprobados: 65,9% con sueño good frente a 44,7% con sueño poor; 67,4% con coaching frente a 45,5% con self-study.

## El modelo de regresión lineal múltiple

- **Multicolinealidad:** no hay. Todos los VIF son prácticamente 1.
- **Selección de variables:** backward, forward y stepwise con AIC llegan al mismo modelo de 9 variables. Salen `internet_access` y `exam_difficulty`.
- **Ajuste:** $F_0 = 110983{,}08$ (p-valor < 0,001), $R^2 = 0{,}7789$, $\hat{\sigma} = 8{,}89$ puntos.

| Variable | Efecto sobre la calificación promedio, manteniendo constantes las demás |
|---|---|
| `study_hours` | +5,68 puntos por cada hora de estudio |
| `class_attendance` | +0,31 puntos por cada punto porcentual de asistencia |
| `sleep_hours` | +1,32 puntos por cada hora de sueño |
| `sleep_quality` | +4,68 (average) y +8,92 (good) frente a poor |
| `study_method` | entre 4,45 y 9,10 puntos menos que coaching |
| `facility_rating` | +3,69 (medium) y +7,28 (high) frente a low |
| `age`, `gender`, `course` | menos de 0,25 puntos |

### Supuestos del modelo

| Supuesto | Prueba | Resultado |
|---|---|---|
| Independencia | Durbin-Watson = 2,001 y ACF | Se cumple |
| Normalidad | Lilliefors, $D = 0{,}0021$ | La prueba rechaza por el tamaño de muestra; el Q-Q plot sigue la recta |
| Media cero | Prueba $t$, p-valor = 1 | Se cumple |
| Varianza constante | Breusch-Pagan = 867,4 | No se cumple en los extremos, porque la nota está acotada entre 19,6 y 100 |

No se encontraron outliers (prueba de Bonferroni) y el modelo no cambia al quitar las observaciones influenciales (distancia de Cook).

### Ejemplo de predicción

Estudiante de b.tech de 21 años, 4 horas de estudio, 72% de asistencia, 7 horas de sueño average, self-study e instalaciones medium:

- Calificación estimada: 59,39
- Intervalo de confianza del promedio (95%): de 59,30 a 59,47
- Intervalo de predicción para una estudiante (95%): de 41,95 a 76,82

## Conclusión

El desempeño académico depende sobre todo de cuántas horas estudia el estudiante y, en menor medida, de su método de estudio, su calidad de sueño, la calidad de las instalaciones, su asistencia a clases y sus horas de sueño. La edad, el género, el programa académico, la dificultad del examen y el acceso a internet no lo explican.

**Limitación:** la varianza de los errores no es constante en los extremos, así que las estimaciones para las calificaciones más bajas y más altas hay que tomarlas con precaución.

## Preguntas que nos pueden hacer

- **¿Por qué pruebas no paramétricas?** Porque Lilliefors rechaza la normalidad de `exam_score` en todos los grupos.
- **¿Por qué Lilliefors y no Shapiro-Wilk?** Shapiro-Wilk solo admite hasta 5.000 observaciones.
- **¿Por qué Spearman en correlación y Pearson en regresión?** Spearman porque las variables no son normales; Pearson en la regresión porque mide la relación lineal, que es la que describe el modelo. Los valores son casi iguales (0,77 y 0,76 para `study_hours`).
- **Si Levene rechaza, ¿por qué seguimos?** Las desviaciones estándar entre grupos difieren en menos de 1 punto. La prueba rechaza por el tamaño de muestra.
- **¿Por qué el modelo conserva `age`, `gender` y `course` si casi no influyen?** Porque el AIC las conserva: con tantos datos salen significativas aunque su efecto sea menor de 0,25 puntos.
- **¿Qué significa el coeficiente de una variable dummy?** La diferencia en la calificación promedio frente a la categoría de referencia (poor, low, coaching, female, b.com).
- **¿Por qué el intervalo de predicción es tan amplio si el de confianza es tan estrecho?** El de confianza es para el promedio, que se estima con 630.000 datos. El de predicción es para un solo estudiante, cuya nota varía unos 8,89 puntos alrededor de ese promedio.
