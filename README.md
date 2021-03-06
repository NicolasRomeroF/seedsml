# Machine Learning con Spark R (utilizando Google Cloud Platform)
## Integrantes:
- Nicolás Romero
- Richard Torti
- Franco Leal

## Descripción del problema
El trigo es un cereal ampliamente aceptado en la dieta humana y es, de hecho, el cereal más cultivado. Naturalmente, existen diferentes variedades de trigo, los cuáles poseen propiedades nutricionales divergentes y, por tanto, su uso es recomendado en diferentes preparaciones.
Es por ello que en el Instituto de Agropecuario de la Academia Polaca de Lublin, se dedican a cultivar diferentes variedades de trigo e investigan qué características geométricas son las que mejor describen a las semillas.
Por lo tanto, el problema a resolver corresponde a cómo diferenciar las distintas variedades de tipos de grano de trigo en base a las medidas propuestas por el Instituto. 
Ante esto, se elabora el laboratorio N°2 el cual consiste en entrenar un modelo de Machine Learning utilizando el lenguaje R en conjunto con Spark, utilizando una base de datos distribuida en los servidores de Google Cloud Platform. Además, se debe disponibilizar un servicio API para consultar el modelo.

## Enfoque de la solución
Para dar solución al problema planteado, se utiliza un dataset de semillas, el cual se utiliza para predecir la clase de nuevas semillas segun su forma geométrica. Estas semillas pueden ser de 3 tipos:
- Kama
- Rosa
- Canadian

Los parámetros que se definen en el dataset son los siguientes:
- area (Área)
- perimeter (Perímetro)
- compactness (Compacidad)
- lengthk (Length kernel - largo del núcleo)
- widthk (Width kernel - ancho del núcleo)
- asymmetry (Asimetría)
- lengthkg (Length of kernel groove - largo de la ranura del núcleo)

El método de clasificación usado para entrenar el modelo fue el de Random forest (Bosques aleatorios) que opera construyendo arboles de decisión en base un vector aleatorio, de esta forma se corrige el hábito de sobre ajustamiento de los árboles de decisión.

## Desarrollo de la actividad

### Inconvenientes

Uno de los principales inconvenientes es la instalación propiamente tal del entorno de ejecución de la solución, para lograr utilizar Spark y posteriormente, montar el servicio para consultar el resultado.

### Funcionalidades principales

1.- Predicted

Funcionalidad que permite realizar una consulta con valores por defecto:
- area: 14.88
- perimeter: 14.57
- compactness: 0.8811
- lengthk (largo del núcleo): 5.554
- widthk (ancho del núcleo): 3.333
- asymmetry: 1.0180
- lengthkg (largo de la ranura del núcleo): 4.596

Se accede mediante la ruta /predicted con método GET.

2.- Predict

Funcionalidad que permite predecir la clase de una nueva semilla. Se deben entregar los parámetros indicados anteriormente.
Se accede mediante la ruta /predict con método POST.

3.- Info

Funcionalidad que entrega la precision del modelo entrenado. Se accede mediante la ruta /info con metodo POST.
La precision del modelo es de 96.78%

## Resultados

Para ejemplificar el uso de la API, se utiliza swagger:

Funcionalidad predicted:  
![alt image](https://i.ibb.co/9tGTkg9/Captura-de-pantalla-de-2019-05-21-00-39-26.png "Ejemplo predicted")

Funcionalidad predict:  
![alt image](https://i.ibb.co/qRhQdTN/Captura-de-pantalla-de-2019-05-21-01-08-13.png "Ejemplo predict")

Clusters del servidor:  
![alt image](https://i.ibb.co/234x9hN/Captura-de-pantalla-de-2019-05-21-00-57-14.png "N° clusters")

El tiempo de respuesta del servidor (3 nodos) para cargar el dataset y entrenar el modelo es de aproximadamente 30 segundos, mientras que en local (1 nodo) demora 4 segundos. Se infiere que esto se debe al alto overhead que sufre la solución dado un dataset pequeño y muchos nodos, ya que en comparación, la comunicación entre nodos es mucho mayor en proporción que la computación de la solución en sí. Sin embargo, si la cantidad de datos creciera, la solución distribuida sería más eficiente, ya que la relación comunicación vs tiempo de procesamiento es ínfima.

## Acceso a producción

Se puede acceder al servidor de la solución [aquí](http://35.247.217.37:4104/).

## Desplegar servicio

Para desplegar el servicio, se requieren los siguientes softwares:

### R
Para instalar R se deben seguir los siguientes pasos (en la terminal):
#### Paso 1:
Agregar GPG key:
```sh
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
```
#### Paso 2:
Agregar respositorio

```sh
$ sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'
```

#### Paso 3:
Actualizar índice

```sh
$ sudo apt update
```

#### Paso 4:
Instalar R


```sh
$ sudo apt install r-base
```

#### Paso 5:
Verificar instalación accediendo a la cosola de R:
```sh
$ sudo -i R
```

Luego, se deben instalar las siguientes librerías para R (una vez dentro de R) con el siguiente comando.
```sh
$ install.packages("<Nombre del paquete>")
```
- readr  
- dplyr  
- plyr 
- plumber
- SparkR

### Spark

Al utilizar Spark en R, se configura automáticamente un entorno con un nodo de Spark. Sin embargo, en nuestro proyecto utilizamos el servicio Google Cloud con un cluster de 3 nodos (1 master y 2 worker). Se deja adjunto archivo Terraform para crear automaticamente un cluster en Google Cloud Dataproc, previamente se debe obtener un archivo JSON con las credenciales de la cuenta de Google Cloud y guardarlas en la misma carpeta del script con el nombre "CREDENTIALS_FILE.json". Ademas se deben tener activadas la API de Google Dataproc en el proyecto.

![alt image](https://i.ibb.co/znMw2TQ/imagen.png "Cluster")
![alt image](https://i.ibb.co/LzTRfL6/imagen.png "Instancias VM")

Alternativamente se deja una carpeta con un archivo docker-compose que permite configurar un entorno Spark en Docker con un nodo master y dos worker (obtenido desde: https://medium.com/@_seraph1/como-correr-apache-spark-desde-una-imagen-docker-88f62c676b2f)

### Funcionamiento

En la consola de R:

#### Paso 1:
Cambiar el directorio de trabajo a donde esta el script
```sh
setwd("/<Path del script>/")
```

#### Paso 2:
Cargar la libreria plumber
```sh
library(plumber)
```

#### Paso 2:
Correr el archivo, este mostrara la API en el puerto 4104 (puede ser cambiado).
```sh
plumb(file='seedsml.R')$run(host="0.0.0.0", port=4104)
```

