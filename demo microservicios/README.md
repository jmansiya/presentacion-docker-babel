# EQUIPO ARQUITECTURA BABEL. 

## DEMO. DOCKERFILE.
En este apartado de la demo explicaremos brevemente la generación de imágenes docker mediante Dockerfiles.

### GENERACIÓN DE IMAGEN A PARTIR DE UN FICHERO DOCKERFILE.

En la demo anterior, estudiamos un método para crear nuevas imágenes a partir de contenedores que anteriormente habíamos configurado. En esta entrada vamos a presentar la forma más usual de crear nuevas imágenes: usando el comando docker buid y definiendo las características que queremos que tenga la imagen en un fichero Dockerfile.

Un **Dockerfile** es un fichero de texto donde indicamos los comandos que queremos ejecutar sobre una imagen base para crear una nueva imagen. El comando docker build construye la nueva imagen leyendo las instrucciones del fichero Dockerfile y la información de un entorno. Tenemos que tener en cuenta que cada instrucción ejecutada crea una imagen intermedia.

![alt text](https://github.com/jmansiya/presentacion-docker-babel/blob/master/demo%20microservicios/dockerfile.png "DockerFile to image")

**Buenas prácticas al crear Dockerfile**

**1 - Los contenedores deber ser “efímeros"**
Cuando decimos “efímeros” queremos decir que la creación, parada, despliegue de los contenedores creados a partir de la imagen que vamos a generar con nuestro Dockerfile debe tener una mínima configuración.

**2 - Uso de ficheros .dockerignore**
Como hemos indicado anteriormente, todos los ficheros del contexto se envían al docker engine, es recomendable usar un directorio vacío donde vamos creando los ficheros que vamos a enviar. Además, para aumentar el rendimiento, y no enviar al daemon ficheros innecesarios podemos hacer uso de un fichero .dockerignore, para excluir ficheros y directorios.

**3 - No instalar paquetes innecesarios**
Para reducir la complejidad, dependencias, tiempo de creación y tamaño de la imagen resultante, se debe evitar instalar paquetes extras o innecesarios Si algún paquete es necesario durante la creación de la imagen, lo mejor es desinstalarlo durante el proceso.

**4 - Minimizar el número de capas**
Debemos encontrar el balance entre la legibilidad del Dockerfile y minimizar el número de capa que utiliza.

**5 - Indicar las instrucciones a ejecutar en múltiples líneas**
Cada vez que sea posible y para hacer más fácil futuros cambios, hay que organizar los argumentos de las instrucciones que contengan múltiples líneas, esto evitará la duplicación de paquetes y hará que el archivo sea más fácil de leer. Por ejemplo:

```javascript
RUN apt-get update && apt-get install -y \
git \
wget \
apache2 \
php5
```

### Creación de imagen con el servidor web apache2 con Dockerfile.

Para ello primero creamos un directorio vacio donde añadiremos un fichero index.html y el fichero Dockerfile para la generación de la imagen:

        $ mkdir apache
        $ cd apache/
        $ echo "<h1>Prueba de funcionamiento contenedor docker</h1>">index.html

Creamos el siguiente fichero Dockerfile en el mismo directorio apache que acabamos de crear:

```javascript
FROM debian
MAINTAINER José Mansilla García-Gil "jose.mansilla@babel.es"

RUN apt-get update && apt-get install -y \
    apache2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2

EXPOSE 80
ADD ["index.html","/var/www/html/"]

ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Creamos la imagen:

        $ docker build -t josemansilla/apache2:1.0 .

Si todo ha salido correctamente tendremos un mensaje de:
 Successfully built f483a08f0b13
 Successfully tagged josemansilla/apache2:1.0

Ahora podemos comprobar si en el repositorio local tenemos la nueva imagen que acabamos de generar:

    $ docker image ls
    
        REPOSITORY             TAG                 IMAGE ID            CREATED              SIZE
        josemansilla/apache2   1.0                 f483a08f0b13        About a minute ago   204MB

Ejecutamos la imagen generada:

    $ docker run -itd -p 80:80 --name servidor_web josemansilla/apache2:1.0


### Creación una imagen con con php a partir de nuestra imagen con apache2
Para ello creamos un nuevo directorio vacio, por ejemplo:

        $ mkdir php  
        $ cd php/
        $ echo "<?php echo phpinfo();?>">index.php

Creamos el siguiente fichero Dockerfile en el mismo directorio apache que acabamos de crear:

```javascript
FROM josemansilla/apache2:1.0
MAINTAINER José Mansilla García-Gil "jose.mansilla@babel.es"

RUN apt-get update && \
    apt-get install -y php && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 80
ADD ["index.php","/var/www/html/"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

        $ docker build -t josemansilla/php7:1.0 .
        $ docker run -d -p 8080:80 --name servidor_php josemansilla/php7:1.0

Para comprobar que todo ha ido correctamente podemos abrir en el navegador la siguiente dirección: *http://localhost:8080/index.php* se mostrará la página en php que creamos en los pasos previos y si ponemos *http://localhost* se mostrará la página index.html que se creo en la primera imagen con apache únicamente.


### Gestionando el registro Docker Hub

## DEMO. MICROSERVICOS SPRING BOOT, SERVICIOS NETFLIX EN DOCKER.
En este directorio tenemos el código de los diferentes servicios de Spring Boot y de la arquitectura Netflix que se utilizarán en la DEMO 3 de la presentación de Introducción a Docker.
