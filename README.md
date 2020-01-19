### README

## Idea
Seleccionar principales zonas turísticas de Madrid para medir distancias entre estas y los apartamentos de Airbnb. Seleccionar aquellos a los que se les pueda incrementar el precio por buena ubicación (buena ubicación = +demanda = +comisión para Airbnb

## Estrategia
Se enviará un reporte mensual en el cual se seleccionarán aquellos pisos a menos de 500m de las principales zonas turísticas de Madrid. Esto indicará que la ubicación es apta para poder ir a pie. Para ello, se utilizarán las herramientas disponibles en la nube (Google Cloud). Posteriormente, y fuera de Cloud, se avisará a los anunciantes de que sus pisos son aptos para una subida de precios.

## Arquitectura
Arquitectura en Google Cloud con scrappy, Google Cloud storage, Google Cloud dataproc, Hive.

Se diseña un crawler con scrappy para recoger la información de Civitatis, esto volcará en un segmento del Google Cloud. El crawler se lanzará con una Cloud Function, en este caso HTTP.

Los datos del dataset de Airbnb también se insertarán en un segmento de Google Cloud. Ambos ficheros (tanto el crawler como el dataset), irán en el mismo. En este caso, se cargará a mano.

Una vez están todos los datos en el segmento “bda-laura1”, se crean 2 tablas en Hive donde se hará el procesamiento y se obtendrá el resultado de los apartamentos más cercanos a las zonas turísticas. 

El resultado de la query será un output en Google Storage. Guardado en el mismo segmento “bda-laura1”.

## Operating model
Se disparará de forma mensual el Cloud Function para actualizar las atracciones turísticas (por si hay novedades). El resultado se guarda en un el segmento gs://bda-laura1/input_civitatis.

Habrá otro input con el dataset de Airbnb guardado de forma manual en este mismo segmento: gs://bda-laura1/input_airbnb
Se levantará el cluster en Google Cloud también de forma manual cada vez que se quiera generar este informe.

Una vez levantado el cluster y para procesar la información:

Ejecución
•	Creación de la tabla Airbnb + carga datos del .csv (Airbnb-listings.csv) a Hive: Se haría con “load data inpath”. Coge los datos del Google storage para cargarlos el Hive.

•	Creación de la tabla civitatis + carga datos del .csv (crawl.csv) a Hive: Se haría con “load data inpath”. Coge los datos del Google storage para cargarlos el Hive.

•	Una vez tenemos las dos tablas creadas nos conectamos a Hive desde el SSH de la rama master del clúster con Beeline (JDBC) para realizar las consultas. 

•	Después, haríamos un Join que nos permita calcular las distancias entre un punto y otro. Los pisos de Airbnb y las zonas turísticas de Madrid.

•	Seleccionaríamos aquellos a una distancia <= a 500m ya que son los que se considera que son aptos para visitar estas zonas a pie. El resultado de esto es que tienen una ubicación “privilegiada” y en ese sentido, los anunciantes tendrían la opción de incrementar el precio de los apartamentos por noche. 

•	Se apaga el cluster para no gastar recursos.

•	Con el output generado y ya fuera de Google Cloud, se haría una comunicación masiva vía email y SMS a los anunciantes. En el caso de que se incrementen los precios de los apartamentos, se incrementaría también la comisión que recibe Airbnb. Resultado: incremento de los beneficios.

## Desarrollo
1.	Creación del Segmento 

•	Subida del fichero a mano
 
•	El crawler entra de forma automática tras activar la función
 


2.	Despliegue la Cloud Function 
https://europe-west1-big-data-architectura-laura.cloudfunctions.net/function-civ
 
 
3.	Levantar el cluster
•	Escogemos el segmento ya creado con los datos.
 
 
•	Resultado cluster configurado
 
4.	Tabla Airbnb
 

•	Create table:
 
•	Load data del Google storage:
 





•	Dentro de la rama master nos conectamos via SSH a Beeline para realizar consultas: 
 
 

5.	Tabla civitatis 

 









•	Create table:
 
•	Load data del Google storage:
 
•	Contenido de la tabla
 

•	Ambas tablas en Hive
 

6.	(Extra) Procesamiento de datos con Wordcount
 
 




•	De esta forma se guardaría el resultado en el hdfs y no en el segmento
 

•	También lo podemos guardar en el mismo segmento si lo preferimos
 

 


•	Fichero output del Google storage
 
 
•	Output: fichero final procesado
 

DIAGRAMA: https://docs.google.com/drawings/d/1J6fkWkJZ9iG3tmM_QKqH9VDGb7dQMmw_cauDeh1plKw/edit?usp=sharing
