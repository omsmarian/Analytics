# Outline de presentación - TP Analytics

**Duración target**: 15-20 min exposición + 5 min Q&A
**Formato**: 10 diapositivas + notebook Jupyter como soporte técnico

---

## Slide 1 - Portada

**Título**: Análisis de cartera y gasto - Prepaga de salud

**Subtítulo**: TP Final Analytics - ITBA - Abril 2026

**Contenido**:
- Nombres del grupo
- Fecha de presentación
- Logo ITBA

**Qué contar de palabra**: "Vamos a presentar un análisis sobre la cartera de una prepaga de salud. El trabajo cubre exploración de datos, modelos predictivos y segmentación, aplicando los conceptos vistos en las cuatro clases."

---

## Slide 2 - Contexto y problema

**Título**: El dataset y lo que busca responder

**Contenido**:
- 6.000 atenciones con internación, 12 columnas
- Datos demográficos + clínicos + administrativos
- Periodo: 2023-2025
- Problema: describir la cartera, analizar el uso del sistema, encontrar diferencias en billing, y ver si hay insights accionables para la prepaga

**Visual sugerido**: Tabla chiquita con las 12 columnas agrupadas en 3 bloques (demográfico / clínico / administrativo) o una sola imagen con el head del CSV

**Qué contar de palabra**: Mencionar el framework de la clase 1 (Problema claro → Datos → Patrón → Acción → Resultado medible) como guía para el análisis.

---

## Slide 3 - Calidad de datos y limpieza

**Título**: Qué encontramos antes de analizar

**Contenido**:
- 1112 registros (~19%) sin medicamento - todos son los "Healthy"
- 30 registros sin condición médica, 20 sin billing
- Gender con casing inconsistente (MALE vs Male)
- **34 registros con edad 0** - incluye bebés con cáncer, hipertensión y obesidad. Claramente error de carga
- Test Results: 51% Abnormal, 38% Normal, 11% Inconclusive

**Visual sugerido**: Dos bloques - izquierda "problemas encontrados" con iconitos, derecha "qué hicimos" (capitalize en gender, flag outliers de billing, imputación de edades imposibles con mediana por condición)

**Qué contar de palabra**: Es importante llegar a esta slide antes de los resultados. Los pibes que vean esto saben que hicimos el trabajo sucio de limpiar antes de modelar. La frase clave: "el 19% sin medicamento no es random - son exactamente los pacientes clasificados como Healthy, lo que sugiere que la variable indica 'internación sin patología activa'".

---

## Slide 4 - Descripción de la cartera

**Título**: Quiénes son los asegurados

**Contenido**:
- Edad media: 51 años. Distribución bastante plana entre 18 y 85
- Género: 52% Femenino / 48% Masculino (casi 50/50)
- Tipo de sangre: coincide con distribución real de la población
- 4 providers (Aetna, Blue Cross, Cigna, Medicare) - repartidos casi 25/25/25/25
- 4 planes (Basic, Silver, Gold, Platinum) - también repartidos de forma pareja

**Visual sugerido**: Dashboard de 4 gráficos chicos: pirámide de edad, donut de género, donut de provider, barra de planes. El dashboard puede ir entero o se pueden embeber desde el notebook (interactivos con plotly)

**Qué contar de palabra**: La cartera está balanceada demográficamente. Lo que no está balanceado es lo que pasa adentro - eso lo vemos en las próximas slides. También decir que los 34 registros con edad 0 los tratamos aparte y mencionar que son outliers evidentes.

---

## Slide 5 - Uso del sistema: condiciones y medicamentos

**Título**: Qué internaciones hay y qué se receta

**Contenido**:
- Top condiciones: Obesidad (23%), Hipertensión (22%), Healthy (18%), Asma (14%), Diabetes (14%), Cáncer (9%)
- Admisiones: 50/50 entre Elective y Emergency
- Pertinencia medicamento-condición: **100% coherente** en los casos con datos completos. Cada patología recibe los medicamentos esperados
- Duración promedio: 8.5 días, rango 3-14 días

**Visual sugerido**: Heatmap del crosstab condición × medicamento (es el más vistoso) + una barra al costado con top condiciones. El heatmap es el que tienen los pibes en el notebook, bloques perfectamente limpios por patología.

**Qué contar de palabra**: "El dataset está limpio clínicamente. No hay asmáticos recibiendo quimioterapia. El problema de la prepaga no es mala prescripción - es volumen de crónicos". Esta frase resume la conclusión principal de la sección.

---

## Slide 6 - Diferencias de billing

**Título**: Dónde está la plata

**Contenido**:
- Billing medio: $3.693. Rango $1.050 - $8.115
- **Por condición**: mediana parecida entre todas (~$3.400). Las condiciones crónicas no cuestan más por caso, pero son muchos más casos
- **Por edad**: aumenta con la edad (mediana $2.800 en 18-30 vs $4.200 en 76+)
- **Por admission type**: Emergency ~50% más caro que Elective
- **Por plan**: **Basic, Silver, Gold y Platinum tienen billing medio similar**. Hay un problema de pricing o diferenciación

**Visual sugerido**: Dos boxplots lado a lado: billing por condición + billing por franja etaria. Agregar un callout/flecha con el insight del plan ("si el básico cuesta lo mismo que el premium, ¿qué segmentamos?")

**Qué contar de palabra**: Lo de los planes es el hallazgo más inesperado. La prepaga probablemente tiene un problema de diseño de producto ahí. Acá también tirar el dato de que Obesidad e Hipertensión no tienen billing medio más alto, pero como hay muchos casos concentran el mayor gasto total (+30% del total entre las dos).

---

## Slide 7 - Modelo de score: predecir alto costo

**Título**: ¿Podemos anticipar qué paciente va a ser caro?

**Contenido**:
- Target: `Alto Costo = 1` si billing > percentil 75 ($4.652)
- Dos modelos en paralelo: **Logistic Regression** vs **Decision Tree Classifier**
- **Resultado**: AUC 0.934 (LR) vs 0.933 (DT) - ambos excelentes
- Si intervenimos solo en el top 30% de score, capturamos ~80% de los alto-costo reales
- Variables más importantes: Admission Type (77%), Edad (23%)

**Visual sugerido**: Curva ROC superpuesta con las dos curvas + tabla chiquita de métricas comparativas. Exactamente el slide de la 4ta clase. Opcional: mini matriz de confusión al costado.

**Qué contar de palabra**: Este es el corazón de la parte de ML. Mencionar explícitamente que seguimos el framework de la clase 4 para comparar modelos. La conclusión práctica: "la prepaga puede intervenir en 3 de cada 10 pacientes y capturar el 80% del gasto evitable. Ahorra 70% del esfuerzo operativo".

---

## Slide 8 - Segmentación con K-medias

**Título**: Cuatro perfiles de paciente

**Contenido**:
Tabla con los 4 clusters:
| Cluster | Edad media | Duración | Billing | % | Lectura |
|---------|-----------|----------|---------|---|---------|
| 0 | 68 años | 6.5 días | $5.716 | 21% | Mayores - internaciones cortas y caras |
| 1 | 34 años | 10.6 días | $2.342 | 26% | Jóvenes - internaciones largas y baratas |
| 2 | 41 años | 6.0 días | $4.020 | 25% | Adultos - internaciones cortas costo medio |
| 3 | 66 años | 10.6 días | $3.084 | 27% | Mayores - internaciones largas costo medio |

**Visual sugerido**: La tabla completa + un scatter 2D (Edad vs Billing) con colores por cluster. O reemplazar la tabla por 4 "cards" de perfil.

**Qué contar de palabra**: "K-medias es no supervisado. No le dimos un target, solo pedimos que encuentre grupos de pacientes parecidos entre sí. El cluster 0 es el de mayor prioridad de intervención - son mayores que gastan mucho en poco tiempo, probablemente emergencias o procedimientos intensivos". Conecta con el concepto de segmentación de la clase 3.

---

## Slide 9 - Conclusiones y recomendaciones

**Título**: Qué le decimos a la prepaga

**Contenido** (3 recomendaciones concretas):

1. **Priorizar prevención en Obesidad + Hipertensión**. Son casi el 45% de la cartera y generan más de un tercio del gasto. Bajar 10% sus internaciones mueve la aguja más que cualquier otra intervención.

2. **Revisar el diseño de planes**. Si Basic y Platinum cuestan lo mismo a la prepaga, hay un problema de pricing o de diferenciación real. Hay plata dejada en la mesa.

3. **Limpiar la carga de datos**. 19% sin medicamento, 34 bebés con patologías crónicas, Gender inconsistente. Son problemas operativos, no analíticos. Cualquier modelo futuro arranca con ruido si no se resuelven.

**Visual sugerido**: 3 cards verticales con icono + título corto + 1-2 líneas de bullet. Que sea súper escaneable.

**Qué contar de palabra**: Estas son las 3 cosas que nos llevamos del análisis. Cada una es accionable. No son "es interesante que..." sino "hay que hacer X".

---

## Slide 10 - Datos que faltan y próximos pasos

**Título**: Lo que no pudimos responder y por qué

**Contenido**:
- **Sin ID de paciente**: no podemos detectar reingresos ni calcular lifetime value. Es la limitación más grande del dataset
- **Sin severidad de la condición**: "Diabetes" a secas no alcanza - no es lo mismo un controlado que uno con complicaciones
- **Sin costos desglosados**: el billing es total. Para decidir dónde recortar necesitamos saber qué parte es medicamento, qué parte es cama, qué parte son estudios
- **Preguntas nuevas**: ¿hay estacionalidad? ¿qué son los 1108 "Healthy" internados? ¿los bebés con crónicos son error?

**Visual sugerido**: Dos columnas: "Lo que tenemos" (tick verde) vs "Lo que nos falta" (cruz roja). O una timeline de "próximas preguntas" si hay tiempo.

**Qué contar de palabra**: Cerrar con honestidad metodológica. Las limitaciones del dataset no invalidan el análisis, pero sí marcan qué va primero en una segunda iteración. Y gracias por escuchar.

---

# Notas generales para armar las slides

**Estilo recomendado**:
- Plantilla minimalista - fondo blanco o gris muy claro
- Una paleta de máximo 3 colores (azul ITBA + gris + un acento)
- Fuente sans-serif, tamaño mínimo 18pt para que se lea atrás
- Un gráfico por slide, máximo dos si son complementarios
- Si una slide necesita más de 5 bullets, partirla en dos

**Durante la presentación**:
- Si preguntan algo técnico (cómo se calculó X, qué devolvió Y), tirar el notebook en vivo
- Tener el notebook abierto en una pestaña por las dudas
- El gráfico interactivo de gastos por enfermedad × edad (widget con slider) es impactante si lo mostrás en vivo

**Tiempo sugerido por slide**:
- Slides 1-3: ~1 min cada una (intro, contexto, limpieza)
- Slides 4-6: ~2 min cada una (análisis descriptivo)
- Slides 7-8: ~3 min cada una (los modelos son el highlight técnico)
- Slide 9-10: ~2 min cada una (conclusiones)
- Total: ~18 min, queda margen para Q&A

**Qué NO poner en las slides pero sí tener de backup**:
- Detalle del preprocesamiento (outliers IQR, imputación de edades)
- Árbol de regresión visualizado (reglas largas)
- Método del codo para elegir K
- Recall acumulado por decil detallado
- Matriz de confusión desglosada

Todo eso está en el notebook. Si alguien pregunta, abrimos el notebook y mostramos.
