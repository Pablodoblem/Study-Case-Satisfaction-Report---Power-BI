## Data Analysis Project: [Título del Proyecto]

## Descripción del Proyecto

Hello world, here I am with one of the latest projects I've been evolved. The study case prensents a company called HS that wants to measure the satisfaction level of their workers. 
In that particular case we will be working with a vbery basic dataset where we'll find some dimentional tables that will be linked and refered to a fact table.
- **Tema**: Análisis del índice de satisfacción de los empleados de la empresa HR
- **Objetivo**: Investigar la relación entre precios y crecimiento demográfico.

---

## Pasos del Proceso

### 1. Recolección de Datos
Como en todo análisis, comenzaremos con la importación de los archivos csv con los que trabajaremos. A continuación mostramos las tablas "Employee", "RatingLevel", "SatisfiedLevel", "EducationLevel" y "PerformanceRating":

![Recolección de datos](images/2.1.png)
![Recolección de datos](images/2.2.png)
![Recolección de datos](images/2.3.png)
![Recolección de datos](images/2.4.png)
![Recolección de datos](images/2.5.png)

Como podemos observar, las tablas están bastante sanas. Tan solo debemos adaptar el tipo de dato de algunas columnas así como el formato de sus celdas. 

### 2. Metodología Kimball:

En esta ocasión trataremos de adoptar el modelo Snowflake ideado por Kimball dado su gran eficiencia y popularidad. Por ello trataremos de buscar qué tablas van a ser consideradas como tablas dimensionales y tabla de hechos y renombraremos cada tabla con el prefijo "Dim-" o "Fact-" en función de su tipología. A continuación se muestra la categorización propuesta:

![Recolección de datos](images/3.png)

### 3. Limpieza de Datos
Una vez finalizada la importanción de las tablas y su renombramiento, nos decidimos a modelar los datos de manera que se muestren de una manera coherente y sólida para luego poder tratar las relaciones en la vista de modelo. Tras revisar los datoss nos encontramos con la necesidad de covertir la columna "ReviewDate" de la table "FactPerformaceRating". Como se puede apreciar observamos que los números de la fecha se ven expresados en ocasiones como un número de una sola cifra. Esto no debería de ser un problema dado que al cambiar de tipo de dato a "Date" debería interpretarlo sin mayor problema. El problema que he experimentado yo es que el formato de fecha según mi pais es dia/mes/año por lo que debo expresar las mismas fechas en una columna personalizada pero de manera ordenada. También aprovecharé esta ocasión para desarrollar un código que poder utilizar en futuras ocasiones y que consiga convertir los números que tan solo tienen una cifra en unos que cuenten con dos añadiendo un 0 delante. 

El código en cuestión está desarrrollado en 
Código utilizado:
```python
import pandas as pd

# Ejemplo de limpieza
df = pd.read_csv('data/housing.csv')
df.dropna(inplace=True)
