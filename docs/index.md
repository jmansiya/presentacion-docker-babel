---
layout: default
title: Demo Comandos Docker
title_menu: Comandos Docker
show_menu: true
weight: 1
---

# {{page.title}}

## Creación de una imagen

Crearemos una imagen nueva a partir de la imagen base de Ubuntu, a la cual le instalaremos un servidor web Apache2.
Lo lanzamos con docker run en modo iteractivo y en segundo plano:

    $ docker run -it --name apache ubuntu /bin/bash
        root@5bb9a0a75180:/# apt-get update    #Actualizamos el sistema
        root@5bb9a0a75180:/# apt-get install apache2
        root@5bb9a0a75180:/# exit


Creamos una nueva imagen a partir del docker que acabamos de ejecutar y al cual hemos instalado un apache2:

    $ docker commit -m "Ubuntu con Apache2 instalado" -a "Jose Mansilla jose.mansilla@babel.es" 5bb9a0a75180 apache2/ubuntu:v1

Acabamos de craer una nueva imagen en nuestro repositorio local.  Comprobamos que existe y que se ha creado correctamente:

    $ docker images
        REPOSITORY                                           TAG                 IMAGE ID            CREATED             SIZE
        apache2/ubuntu                                       v1                  da256f84f390        34 seconds ago      221MB

    $ docker run -it --name apache2 apache2/ubuntu:v1 /bin/bash
        root@703da3340fcc:/# dpkg –l | grep apache2

## Detached o Background. 
Nos permite ejecutar un contenedor en segundo plano y poder correr comandos sobre el mismo en cualquier momento mientras esté en ejecución. Para ello usamos el flag –d.

    $ docker run -d --name apacheServer apache2/ubuntu:v1 /usr/sbin/apache2ctl -D FOREGROUND
    $ docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
        5ba29e6c20dc        apache2/ubuntu:v1   "/usr/sbin/apache2ct…"   3 seconds ago       Up 2 seconds                            apacheServer


    $ docker top apacheServer
    $ docker exec –it apacheServer bash
    root@5ba29e6c20dc:/# exit
    $ docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
        5ba29e6c20dc        apache2/ubuntu:v1   "/usr/sbin/apache2ct…"   2 minutes ago       Up 2 minutes                            apacheServer


## Mapeo de puertos
Para que la aplicación que nos está sirviendo nuestro contenedor sea visible al exterior tenemos que mapear los puertos.  Podemos hacer la redirección de puertos con las opciones:
    -p: especificamos a qué puerto queremos la redirección.
    -P: le dice a Docker que si expone algún tipo de puerto haga el forwarding a un puerto aleatorio. (Dockerfile)

    $ docker run -d -p 8000:80 --name apacheServer2 apache2/ubuntu:v1 /usr/sbin/apache2ctl -D FOREGROUND
    $ docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES  
        a5e0a6d83f61        apache2/ubuntu:v1   "/usr/sbin/apache2ct…"   5 seconds ago       Up 4 seconds        0.0.0.0:8000->80/tcp   apacheServer2

 Ahora si probamos en el navegador la url http://localhost:8000  se mostrará la página web de inicio de Apache2.

## Links
Para que dos contenedores colaboren entre ellos podemos abrir puertos y que se comuniquen por ahí, pero Docker nos ofrece la posibilidad de crear links entre contenedores.

    $ docker run -d --name links01 apache2/ubuntu:v1 /usr/sbin/apache2ctl -D FOREGROUND
//Como se puede ver no exponeos puerto por lo tanto no será accesible.
    $ docker run -d --name links02 --link links01 apache2/ubuntu:v1 /usr/sbin/apache2ctl -D FOREGROUND
// Este tunel es unidireccional 

    $ docker exec links02 cat /etc/hosts
        127.0.0.1	localhost
        ::1	localhost ip6-localhost ip6-loopback
        fe00::0	ip6-localnet
        ff00::0	ip6-mcastprefix
        ff02::1	ip6-allnodes
        ff02::2	ip6-allrouters
        172.17.0.2	links01 b1a58a8cf0e5   <---->
        172.17.0.3	ca9d184b048b

## Volúmenes
Podemos montar volúmenes de datos en nuestros contenedores. Con el flag –v podemos montar un directorio dentro de nuestro contenedor. Así podemos incorporar datos que estarían disponibles para nuestro contenedor. Además seguirá apareciendo después de un reinicio del contenedor, serán persistentes.
También se pueden compartir volúmenes de datos entre contenedores.

    $ docker run -ti --name volumen1 -v /volumen01/  apache2/ubuntu:v1 /bin/bash
    //Si ejecutamos ls -l veremos que existe un volumen con el nombre volumen01

Podemos crear un fichero dentro de ese directorio:
    /volumen01# echo "hola Mundo" >> hola.txt 

    Si salimos del contenedor y arrancamos (docker start volumen1) podremos volver a entrar (docker attach idContenedor) y veremos que el fichero hola.txt sigue existiendo.

    . Create a volume:
        $ docker volume create my-vol
    
    . List volumes:
        $ docker volume ls

    . Inspect a volume:
        $ docker volume inspect my-vol
        {
            "Driver": "local",
            "Labels": {},
            "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
            "Name": "my-vol",
            "Options": {},
            "Scope": "local"
        }

    //La ruta hace referencia a la ruta dentro de la VM de docker.

    . Eliminar un volumen dado
        $ docker volume rm my-vol

    . Podemos eliminar todos los volúmenes que no estén siendo utilizados de un único comando:
        $ docker volume prune

    . Volumen bind o conectados: Es una manera de asociar una carpeta de nuestro equipo real y mapearla como una carpeta dentro de un contenedor.

        $ docker run -d -it --name apache2volumen -v "/Users/josemansilla/Desktop/Presentaciones Trabajo/Presentaciones Babel/mysql2":/var/lib/mysql ubuntu:17.10

        En este caso la carpeta local de nuestro equipo '/Users/josemansilla/Desktop/Presentaciones Trabajo/Presentaciones Babel/mysql2' se mapeará a la carpeta del contenedor '/var/lib/mysql'

        Si hacemos docker inspect apache2volumen nos mostrará los volumenes que tenemos montados.

## Configuración de RED.        
 Para ver los tipos de red disponibles por defecto en Docker, sólo hay que ejecutar:
        
        $ docker network ls
            NETWORK ID          NAME                DRIVER
            eb2291dd0b63        bridge              bridge              
            b9e68dd550d4        host                host                
            ccc5be97afaf        none                null               

- **host** Representa la red del propio equipo y haría referencia a eth0. El contenedor usará el mismo IP del servidor real que tengamos.
- **bridge** representa la red docker0 y a ella se conectan todos los contenedores por defecto.
- **none** siginifica que el contenedor no se incluye en ninguna red y si verificamos esto con el comando ifconfig dentro del contenedor veríamos que solo tiene interfade de loopback lo.

Se pueden ver o inspecionar los detalles de cada red usando:

        $ docker network inspect bridge

### Red Bridge

Puede albergar a 65534 contenedores. Tras ejecutar el comando anterior también podremos ver si esta red tiene o no tiene contenedores en funcionamiento en la red. Todos los contenedores dentro de esta red pueden verse los unos a los otros.

Esto se puede comprobar si ponemos en funcionamiento dos contenedores, por defecto los creará en la red BRIDGE, comprobamos que están en la red con *'docker network inspect bridge'*. Accedemos a uno de ellos *'docker exec -it <id_container> bash'*, instalamos el comando ping si no está instalado en el contenedor (Para ubuntu - apt-get install iputils-ping). Hacemos ping a la ip asignada al otro contenedor que está dentro de la red BRIDGE, los paquetes enviados por el comando ping deben de llegar todos perfectamente.

### Redes definidas por nosotros

Para poder aislar contenedores es necesario definir una red para esos contenedores que sea distinta de la red bridge que se asigna por defecto. Para ello:

        $ docker network create --driver bridge red-aislada

            $ docker network ls
                NETWORK ID          NAME                DRIVER              SCOPE
                fc93cf4cde33        bridge              bridge              local
                ac43dacd13b3        host                host                local
                95b2ffbe0be2        none                null                local
                **d4e4fbf10a2d        red-aislada         bridge              local**
            
Vamos a crear un contenedor nuevo y lo vamos a añadir a la red que acabamos de crear:

        $ docker run -itd -p 8000:80 --net=red-aislada --name apacheServer2 apache2/ubuntu:v1 /usr/sbin/apache2ctl -D FOREGROUND
        
El nuevo contenedor me lo ha creado en otra red y por lo tanto tiene una ip que no será accesible desde los contenedores de la red bridge. (Probar mediante el comando ping)

*Redes superpuestas o OVERLAY network*
Cuando necesitamos añadir mas hosts. Una red superpuesto sirve para unir varios host de una manera segura utiliando VXLAN. Esta librería perita hacer esto, incluso permite desarrollar nuestros propios drivers. 

Para desarrollo de aplicaciones, testing o integración continua nos servirá perfectamente con la redes que vienen creadas por defecto. Sin embargo si queremos definir más de un servicio en un host o tener un enjambre de host para tener un scalamiento horizontal necesitaremos definir este tipo de redes y conviene conocer estas opciones.

Para profundizar en la teoria de red de docker este blog con estas dos entradas es bastante interesante:
- [Don Docker Introducción redes con Docker](http://dondocker.com/como-hacer-redes-con-docker/)
- [Don Docker. Redes con varios host con Docker](http://dondocker.com/redes-con-varios-host-con-docker/)

  
## Variables de entorno
Podemos modificar las variables de entorno de un contenedor con el flag –e (--env). También podemos pasarlas desde un fichero externo con --env-file

Para poder ver las variables de entorno usaremos:
 
        $ docker run -d -p 8000:80 -e ENTORNO='DESARROLLO' --name apacheServer2 apache2/ubuntu:v1 /usr/sbin/apache2ctl -D FOREGROUND
        $ docker exec apacheServer2 env
            PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
            HOSTNAME=62d8f3b8dca7
            ENTORNO=DESARROLLO   
            HOME=/root

        //Podemos pasar las variables en un fichero txt por ejemplo:
        // variables.txt
        //      var1=variable1
        //      var2=variable2
        //      var3=variable3
        //      var4=variable4

        $ docker run -d -p 8000:80 --env-file ./variables.txt --name apacheServer2 apache2/ubuntu:v1 /usr/sbin/apache2ctl -D FOREGROUND
        $ docker exec apacheServer2 env
            PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
            HOSTNAME=ed65ba12ba30
            var1=variable1
            var2=variable2
            var3=variable3
            var4=variable4
            HOME=/root

## Eliminar un contenedor.
Para eliminar un contenedor podemos hacerlo por el nombre o por el ID.  Para poder eliminarlo debe estar parado. Si no lo estuviera tendríamos que pararlo con ‘stop’.
Utilizamos el commando **rm**. Si queremos ahorrarnos el paso de pararlo usamos *–f*.

        $ docker stop apacheServer2
        $ docker rm apacheServer2

## Comandos interesantes.
- docker cp. Permite copiar ficheros desde el host hasta el contenedor y viceversa
        //Tenemos el fichero prueba.txt lo vamos a enviar dentro del tmp del contenedor apacheServer2
        $ docker cp ./prueba.txt apacheServer2:/tmp/prueba.txt

        //Creamos un fichero dentro del tmp del contenedor apacheServer2 y lo enviamos a la máquina host en la que está corriendo el contenedor.
        $ docker cp apacheServer2:/tmp/pruebaContainer.txt ./pruebaContainer.txt

- docker system. Módulo de docker que nos permite conocer el estado del sistema docker.

| COMANDO  |     DESCRIPCION  |
|----------|-------------------|
| docker system df |  Muestra el uso de disco de docker. Numero de imagenes, contenedores, volúmenes así como el tamaño que ocupa |
| docker system events | Obtiene los eventos en tiempo real que se producen desde el servidor |
| docker system info | Muestra información más detallada del sistema |
| docker system prune | Elimina los datos que no están en uso |



        $ docker system df
            TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
            Images              5                   1                   430.1MB             293MB (68%)
            Containers          1                   1                   464B                0B (0%)
            Local Volumes       0                   0                   0B                  0B
            Build Cache         0                   0                   0B                  0B
