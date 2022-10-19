# **_portfolio Mathieu Debrabander_**
## EasyMoney, una entidad financiera con dificultades económicas solicita:
> - un dashboard en Power BI para ver los insights fácilmente
> - una propensión de compra de los productos de mi elección para los clientes existentes
> - una segmentación de los clientes para conocerlos mejor
#### dataset original:
> - 3 archivos de atributos de clientes con 6 millones de registros repartidos en 17 particiones (una por mes). Los atributos son: edad, activo, salario, segmento, canal de entrada, país, región, genero, fallecido y los productos dentro de su cartera.
> - 1 archivo de las 240.000 ventas de 17 meses 
> - 1 archivo con la codificación de los productos
***
***
### 1. dashboard en Power BI
pdf del dashboard completo: [pdf_dashboard_Power_BI](/pdf_dashboard_Power_BI.pdf)  

En cada pestaña se puede filtrar:
> - año y mes
> - margen neto, cantidades vendidas o número de clientes
> - nuevos clientes, clientes existentes o todos 
  
visualización de la página de inicio:  
![Alt text](/foto_pagina1_dashboard.jpg)

Son 7 pestañas: 
> - general: KPIs generales en volumen y variación interanual
> - cliente: atributos de cliente (segmento, edad media por producto, cantidad de clientes por producto, país, salario, genero)
> - mapa de España con los KPIs por provincia, en volumen y variación interanual
> - top 1 cliente: focus en los KPIs del producto con más clientes del momento
> - producto: KPIs de los productos, en volumen y variación interanual y la proporción respectiva de cada producto respecto al total
> - top 1 ventas: focus los KPIs del producto con mayor margen neto del momento
> - alertas: focus en los 5 productos con peores resultados respecto al año anterior, en volumen y procentaje
***
***
### 2. propensión de compra de clientes existentes 
**notebook:** [propension_compra.ipynb](/propensión_compra.ipynb)  

**reto:** crear un dataset de entrenamiento que recoja los atributos de los clientes del mes anterior a la compra. Solo se actualizan los datos un vez al mes. De no hacerlo así, se correría el riesgo de entrenar a tiempo futuro (los atributos del cliente puede variar durante el mes de la compra) y de entrenar con nuevos clientes. 
  
  
Se escogen 4 productos: el top ventas y las 3 peores alertas.    

Después de la limpieza de datos, se realiza un EDA para ver como cambia la distribución de la población según la variable.  
Por ejemplo, se muestra el KDE de la edad de los clientes que compran el producto corto plazo vs la edad de los que no compran.  
También, para cada producto, se analizan las variables más importantes para el modelo.

![Alt text](/kde_age_short_term.jpg)
![Alt text](/feature_importance_long_term.jpg)

Para cada producto, se entrenan 6 modelos: 
> - Support Vector Machine
> - Regresión logística
> - Decision tree classsifier
> - Random forest
> - Gradient boosting
> - XGBClassifier  
  
Se escoge el mejor modelo de base. Se retrabaja el preprocessing, creando nuevas variables y eliminando variables que aportan poco al modelo para ciertos productos. Después se hiperparametriza el modelo de cada producto.  
El resultado es el siguiente:  

|**PRODUCTO**|**ALGORITMO**|**AUC SCORE**|
|--------|--------|--------|
|plan de pensión|XGBClassifier|0,80920|
|inv. corto plazo|Regresión logística|0,95427|
|inv. largo plazo|XGBClassifier|0,87688|
|fondo de inv.|Regresión logística|0,88856|  

**Predicción:** se realiza la predicción con el comando `.predict_proba()` que devuelve la probabilidad de compra para cada cliente en lugar de una clasificación binaria de 0 o 1.  
Por ejemplo, el resultado para el producto inv. corto plazo es así:  

|**ID_CLIENTE**|**PROPENSIÓN DE COMPRA**|
|--------|--------|
|1421783|0,998649|
|1409335|0,998569|
|1440606|0,998248|
|1452072|0,998209|
|1398913|0,99817|
|etc...|etc...| 

**potencial de margen neto**: para calcular el potencial de margen neto por cliente, se multiplica al margen neto medio del producto por la precisión del modelo y por la propensión de compra del cliente. El area de marketing solo tiene que definir cuantos clientes quiere contactar o cual es el umbral de propensión que desae según el presupuesto. Se le puede dar el número de clientes por encima del porcentaje definido y el potencial de margen neto para esos clientes.  
Este es el resultado para los clientes con una propensión de compra mayor de 90%:  

![Alt text](/potencial_por_producto.jpg)

***
### 3. segmentación de la clientela
**notebook:** [segmentacion_clientela.ipynb](/segmentación_clientela.ipynb) 

**reto:** segmentar los clientes en un número coherente de clústers para identificarlos mejor y definir acciones comerciales apropiadas.  

Después de la limpieza de datos, gestión de nulos y feature_engineering se crea la variable 'alta_navidad' porque se aprecia una cierta ciclicidad en la antigüedad.  
Se plotea las variables intrinsecas para ver cuales son las que aportan más valor a modelo:  

![Alt text](/explained_variance.jpg)  

Para determinar el número optimo de clústers, uso el metodo del codo de la libreria yellowbrick: `KElbowVisualizer()`.  
6 es el número optimo de clústers:  


![Alt text](/kmeans_elbow.jpg) 
