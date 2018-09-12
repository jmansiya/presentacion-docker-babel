---

layout: default
title: Demo Docker-Compose
show_menu: true
weight: 4
title_menu: Docker-compose

---

# {{page.title}}

Una vez que ya sabemos como se pueden crear, levantar y distribuir imagenes docker. El siguiente paso es poder levantar un sistema compuesto por varios contenedores que se comunican entre sí. Para ello podemos construir y levantar los contenedores uno a uno o bien, crearnos un ficher **yaml** que pueda ser interpretado por el servicio docker-compose que viene instalado junto con el cliente docker que hayamos instalado en nuestra máquina. 

Para comprobar que tenemos instalado y funcionando correctamente docker-compose:

	$ docker-compose --version

Docker-compose manipula varios contenedores haciendo uso de archivos de configuración en formato YAML. Docker-compose es capaz de leer e interpretar ficheros YAML en los cuales se componen de una serie de pares Key:Value. La lista de posibles keys soportadas por docker-compose se puede ver [aquí](ClavesDockerCompose.html).

Docker-compose soporta una serie de comandos de los cuales los más habituales se pueden ver en este [enlace](ComandosDockerCompose.html).



## MICROSERVICOS SPRING BOOT, SERVICIOS NETFLIX EN DOCKER.

Para realizar una prueba mas completa para poder ver el funcionamiento del comando docker-compose, vamos a realizar el despliegue de un entorno completo formado por varios micro servicios de Spring Boot orquestados por los servicios Eureka y Zuul de la arquitectura Netflix. Además tendremos una SPA desarrollada en Angular que utilizará estos servicios para insertar y consultar información. La arquitectura montada sería la siguiente:

<< PENDIENTE >>

El fichero yaml de nuestro docker-compose es el siguiente:

```yaml
version: '2'
services:
  eurekaservice:
    image: josemansilla/microservice_registration:0.0.1-SNAPSHOT
    restart: always
    ports:
      - 1111:1111
  mongodb:
    image: mongo
    restart: always
  microservice-employee:
    image: josemansilla/microservicio-employee:1
    restart: always
    links:
      - eurekaservice:eureka-docker
      - mongodb
    environment:
      - SPRING_PROFILES_ACTIVE=docker
  microservice-empresa:
    image: josemansilla/microservcio_empresa:0.0.1
    restart: always
    links:
     - eurekaservice:eureka-docker
     - mongodb
    environment:
     - SPRING_PROFILES_ACTIVE=docker
  loadbalancer:
    image: josemansilla/edgeservice:0.0.1
    restart: always
    links:
      - eurekaservice:eureka-docker
    ports:
      - 8085:8085
    environment:
     - SPRING_PROFILES_ACTIVE=docker
  consolaAdministracion:
    image: josemansilla/consola-administracion:0.0.1
    restart: always
    links:
      - eurekaservice:eureka-docker
    ports:
      - 8099:8099
    environment:
     - SPRING_PROFILES_ACTIVE=docker
```

Comprobar que están accesibles los servicios API levantados con Postman y con la consola de administración que levantamos también con el docker-compose.

En esta demo he utilizado contenedores que estan creados previamente y subidos en Docker Hub para su libre distribución. El código de estos servicios Spring Boot se puede encontrar [aquí](https://github.com/jmansiya/microserviciosDockerKubernete).