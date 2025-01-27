# Data Labs Project: [Estado de los empleados]

## Descripción del Proyecto

En el equipo directivo de la compañia de Atlas Lab buscan tener mayor visibilidad sobre las métricas relativas a la satisfacción y estado de sus empleados. Es por ello por lo que nos piden conocer el desgaste de la empresa. Para ello debemos explorar los datos disponibles.

**Tema**: Análisis del índice de satisfacción de los empleados de la empresa HR

**Objetivos**:
  - Conocer la tasa de empleados activos e inactivos con respecto al total registrado.  
  - Analizar sus tendencias de contratación a lo largo del tiempo para ver dónde se halla el mayor crecimiento de nuevos empleados asi como su índice de desgaste o "atrittion"
  - Conocer qué tipo de funciones están contratando dento de la organización para poder planificar futuras necesidades de contratacion

---

## Pasos del Proceso

## 1. Recolección de Datos

Como en todo análisis, comenzaremos con la importación de los archivos csv con los que trabajaremos. A continuación mostramos las tablas "Employee", "RatingLevel", "SatisfiedLevel", "EducationLevel" y "PerformanceRating":

![Recolección de datos](images/2.1.png)
![Recolección de datos](images/2.2.png)
![Recolección de datos](images/2.3.png)
![Recolección de datos](images/2.4.png)
![Recolección de datos](images/2.5.png)

Como podemos observar, las tablas están bastante sanas. Tan solo debemos adaptar el tipo de dato de algunas columnas así como el formato de sus celdas. 

---


## 2. Limpieza de Datos

Una vez finalizada la importanción de las tablas y su renombramiento, nos decidimos a modelar los datos de manera que se muestren de una manera coherente y sólida para luego poder tratar las relaciones en la vista de modelo. Tras revisar los datoss nos encontramos con la necesidad de covertir la columna "ReviewDate" de la table "FactPerformaceRating". Como se puede apreciar observamos que los números de la fecha se ven expresados en ocasiones como un número de una sola cifra. Esto no debería de ser un problema dado que al cambiar de tipo de dato a "Date" debería interpretarlo sin mayor problema. El problema que he experimentado yo es que el formato de fecha según mi pais es dia/mes/año por lo que debo expresar las mismas fechas en una columna personalizada pero de manera ordenada. También aprovecharé esta ocasión para desarrollar un código que poder utilizar en futuras ocasiones y que consiga convertir los números que tan solo tienen una cifra en unos que cuenten con dos añadiendo un 0 delante. 

A continuación podemos observar cómo las fechas no tienen el formato deseado.
Muestra de formato no deseado:

![Recolección de datos](images/5__M_code_new_column.png)

#### ¿Cómo lo solucionamos?
Creemos una nueva columna personalizada con el siguiente codigo M dentro del editor de Power Query. 
El código en cuestión está desarrrollado en DAX

Código que hemos empleado:
```DAX
let
    // Divide and store the date in a small text array using "/" as a delimiter
    Parts = Text.Split([ReviewDate], "/"),
    
    // Make you sure that each part has two digits
    Day = Text.PadStart(Parts{1}, 2, "0"),
    Month = Text.PadStart(Parts{0}, 2, "0"),
    
    // Use the original year format
    Year = Parts{2}
in
    // Combine the parts in local data format "dd/MM/yyyy"
    Day & "/" & Month & "/" & Year

```
Con este código, a pesar de estar ya comentado dentro del mismo, observamos cómo dividimos la cadena de texto de ReviewDate (columna orginal) en un array de strings donde se ha designado como delimitador o separador el caracter "/" de tal forma que podamos almacenar cada substring a distintas variables como "Day", "Month" o "Year". Además el contenido de estas variabless va a poder contar con dos caracteres en caso de haber capturado tan solo uno. Finalmente se procede a capturar como valor resultante la conjunción de las subsstrings almacenadas en las variables de la función de manera que podremos obtener el formato de fecha adaptado al formato europeo.

Columna personalizada con datos en formato europeo:

![Recolección de datos](images/6_Data_dataforma_depurated.png)

---

## 3. Metodología Kimball

En esta ocasión trataremos de adoptar el modelo Snowflake ideado por Kimball dado su gran eficiencia y popularidad. Por ello trataremos de buscar qué tablas van a ser consideradas como tablas dimensionales y tabla de hechos y renombraremos cada tabla con el prefijo "Dim-" o "Fact-" en función de su tipología. A continuación se muestra la categorización propuesta:

![Recolección de datos](images/3.png)

#### 3.1 Tablas adicionales

Como es práctica común haremos inclusión de una tabla de fechas. Para ello hemos importado mediante el siguiente código la tabla DimDate al modelo. Gracias a esta tabla podremos relacionar todas las fechas dispuestas en la tabla de hechos con la tabla DimDate pudiendo manejar así los datos de una manera limpia y ordenada. 

El código en cuestión está desarrrollado en DAX
Código utilizado:
```DAX

DimDate = 
VAR _minYear = YEAR(MIN(DimEmployee[HireDate]))
VAR _maxYear = YEAR(MAX(DimEmployee[HireDate]))
VAR _fiscalStart = 4 

RETURN
ADDCOLUMNS(

    CALENDAR(

                DATE(_minYear,1,1),

                DATE(_maxYear,12,31)

),

"Year",YEAR([Date]),
"Year Start",DATE( YEAR([Date]),1,1),
"YearEnd",DATE( YEAR([Date]),12,31),
"MonthNumber",MONTH([Date]),
"MonthStart",DATE( YEAR([Date]), MONTH([Date]), 1),
"MonthEnd",EOMONTH([Date],0),
"DaysInMonth",DATEDIFF(DATE( YEAR([Date]), MONTH([Date]), 1),EOMONTH([Date],0),DAY)+1,
"YearMonthNumber",INT(FORMAT([Date],"YYYYMM")),
"YearMonthName",FORMAT([Date],"YYYY-MMM"),
"DayNumber",DAY([Date]),
"DayName",FORMAT([Date],"DDDD"),
"DayNameShort",FORMAT([Date],"DDD"),
"DayOfWeek",WEEKDAY([Date]),
"MonthName",FORMAT([Date],"MMMM"),
"MonthNameShort",FORMAT([Date],"MMM"),
"Quarter",QUARTER([Date]),
"QuarterName","Q"&FORMAT([Date],"Q"),
"YearQuarterNumber",INT(FORMAT([Date],"YYYYQ")),
"YearQuarterName",FORMAT([Date],"YYYY")&" Q"&FORMAT([Date],"Q"),
"QuarterStart",DATE( YEAR([Date]), (QUARTER([Date])*3)-2, 1),
"QuarterEnd",EOMONTH(DATE( YEAR([Date]), QUARTER([Date])*3, 1),0),
"WeekNumber",WEEKNUM([Date]),
"WeekStart", [Date]-WEEKDAY([Date])+1,
"WeekEnd",[Date]+7-WEEKDAY([Date]),
"FiscalYear",if(_fiscalStart=1,YEAR([Date]),YEAR([Date])+ QUOTIENT(MONTH([Date])+ (13-_fiscalStart),13)),
"FiscalQuarter",QUARTER( DATE( YEAR([Date]),MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1,1) ),
"FiscalMonth",MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1
)

```

![Recolección de datos](images/7.1_looklike.png)

#### 3.2 Data Model

A continuación presentamos la vista de modelo que ofrece la aplicación de Power BI donde tan solo con la importación se ha creado automáticamente una relación activa. Esta relación conecta la columna "DimEmployee" y "FactPerformanceRating" mediante la columna "EmployeeID". Esto se debe a que Power BI trata de hacer relaciones automáticas lo cual puede presentar problemas si no se hacen las relaciones de la manera adecuada. En este caso no ocasiona ningún conflicto sino lo contrario. Aun así quedan aún muchas relaciones por hacer para poder conformar el modelo de SnowFlake. 

Lo primero, dado que contamos con una tabla de fechas estableceremos relación entre la columna "RevDate" (anteriormente creada) con "Date" de la tabla "DimDate".

![Recolección de datos](images/_connectin_dates.png)

Aprovechando que estamoa trabajando con la tabla DimDates vamos a establecer relación entre "HireDate" de la tabla "DimEmployee" y "Date" de la tabla dimensional de fechas.

![Recolección de datos](images/10_conectando_con_hire_date.png)

Como se puede observar, las tablas que cuentan con un mayor numero de columnas son "DimEmployee", tabla donde se expresan todos los datos y características de cada uno de los trabajadores de la empresa y la tabla de hechos "FactPerformanceRating" donde se recoge el registro de puntuaciones de satisfacción del empleado. Las demás tablas son dimensionales y en su mayoría actuan como leyenda para poder interpretar el valor numérico en distintos campos.

Así se conoce el modelo "Snowflake", se deben encontrar relaciones entre la tabla de hechos y las distintas tablas relacionales. Como apunte me gustaría resaltar que para que esto se pueda dar debe de haberse realizado un modelado de los datos en caso de no poder contar con las talbas dimensionales formateadas para tal propósito. Es decir, frecuentemente el cliente cuenta tan solo con la tabla de hechos de la cual habría que extraer los datos en tablas dimensionales distintas con el propóstio de hacer el modelo muchoo más eficiente tanto en cuestiones de velocidad como de almacenaje. Moviéndonos de nuevo a nuestro caso, como decía, contamos con las tablas dimensionales ya prediseñadas así que nos pondremos manos a la obra con las relaciones.

La tabla de hechos, como pudimos abservar, recoge las puntuaciones de satisfacción de los distintos empleados de la empresa. Es por ello por lo que, vamos a tener que relacionar estas columnas con la tabla de "DimSatisfiedLevel", concretamente con la columna "SatisfactionLevel". Por tranto, las colummnas de la tabla de hechos "JobSatisfaction", "ManagerRating", "RelationshipSatisfaction", SelfRating" y "WorkLifeBalance" deben de apuntar a dicha columna.

![Recolección de datos](images/12_satisfactionleve_different_relationships.png)

En Power BI tan solo puede tener activa una de estas relaciones. Este hecho no quiere decir que no podamos usar las relaciones creadas entre otras columnas pero deben de ser tratadas mediante otros métodos como el del uso de la funcion en DAX "USERRELATIOSHIP" que veremos más adelante. 

Otra relación que podemos crear es entre las columnas "EducationLevel" de "DinEducationLevel" y "Education" de "DimEmployee"

![Recolección de datos](images/11_education_level_connected.png)

FInalmente y a continuacion podemos observar cómo ha quedado nuestro modelo de datos:

![Recolección de datos](images/12.1_datamodel_v1.png)

---

### 4. Medidas

Comencemos a pensar en qué medidas vamos a necesitar realizar. Una vez habiendo 

---

### 5. Primeros objetos visuales


