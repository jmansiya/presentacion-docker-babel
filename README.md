# EQUIPO ARQUITECTURA BABEL. 
# INTRODUCCIÓN A DOCKER.
# DEMOS
Repositorio con los proyectos y con los datos para las demos de la presentación de introducción a Docker del equipo de arquitectura de Babel.

# 1era DEMO. DOCKER-MACHINE
En esta demo veremos la ejecución de algunos comandos de docker-machine (Dependiendo del SO que tengamos necesitaremos instalar una utilizando Virtual Box, para poder hacer funcionar en nuestros equipos docker):

    . Crear una nueva máquina de docker.
        $ docker-machine create --driver virtualbox pruebas   --Si tenemos windows 10 con hyperv el driver sería: 'hyperv'
        $ docker-machine ls
        
        AME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
        pruebas   -        virtualbox   Running   tcp://192.168.99.100:2376           v18.06.0-ce   

        $ docker-machine stop pruebas
        $ docker-machine start pruebas        
        $ docker-machine ip pruebas
        $ docker-machine upgrade pruebas  -- Actualiza versión de Docker.
        $ docker-machine inspect [options] pruebas
        $ docker-machine ssh pruebas
                                    ##         .
                            ## ## ##        ==
                        ## ## ## ## ##    ===
                    /"""""""""""""""""\___/ ===
                ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
                    \______ o           __/
                        \    \         __/
                        \____\_______/
            _                 _   ____     _            _
            | |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
            | '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
            | |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
            |_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
            Boot2Docker version 18.06.0-ce, build HEAD : 1f40eb2 - Thu Jul 19 18:48:09 UTC 2018
            Docker version 18.06.0-ce, build 0ffa825
            docker@pruebas:~$ 

    Para poder conectar el cliente docker al docker engine que se está ejecutando dentro de la máquina virutal ejecutar:

        $ docker-machine env pruebas

            $ docker-machine env pruebas
                export DOCKER_TLS_VERIFY="1"
                export DOCKER_HOST="tcp://192.168.99.100:2376"
                export DOCKER_CERT_PATH="/Users/josemansilla/.docker/machine/machines/pruebas"
                export DOCKER_MACHINE_NAME="pruebas"
                # Run this command to configure your shell: 
                # eval $(docker-machine env pruebas)

    Si ejecutamos: $ eval $(docker-machine env pruebas) cambiaremos de máquina docker. 
    
    Para comprobar que hemos cambiado correctamente de máquina podemos tener antes de ejecutar el comando eval(...) ejecutandose un contenedor en la máquina docker original de nuestro equipo:
        $ docker run -d -it --name nginxTest -p 8080:80 nginx
    
    Con este comando pondremos en funcionamiento un servidor nginx en nuestra máquina docker original. Si ejecutamos *docker ps* podremos ver que el contenedor está corriendo correctamente. Si abrimos un navegador con la url http://localhost:8080 veremos la página de index de los servidores nginx.
    
    Ahora que tenemos un contenedor corriendo en la máquina docker original, ejecutamos el comando eval(..) y ejecutamos **docker ps** veremos que ahora no existe ningún contenedor corriendo en esta máquina. 

    Si ejecutamos veremos que el nombre de la máquina es pruebas la que acabamos de cambiar:
        $ docker system info
            **Containers: 1**
            Running: 0
            Paused: 0
            Stopped: 2
            Images: 1
            ...
            Name: *pruebas*
            ...
            Labels:
            *provider=virtualbox*
            ....

    Si deseamos volver a la máquina docker original en la que estabamos previamente trabajando deberíamos ejecutar el siguiente comando:
        $ eval "$(docker-machine env -u)"

    Ahora si ejecutamos docker ps veremos que seguimos teniendo el contenedor de nginx que habíamos lanzado antes de cambiar de máquina docker y si ejecutamos docker system info, tendremos la información de la máquina docker original. 

# 2da DEMO. COMANDOS DOCKER. CREACIÓN DE UNA IMAGEN.

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


# Detached o Background. 
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


# Mapeo de puertos
Para que la aplicación que nos está sirviendo nuestro contenedor sea visible al exterior tenemos que mapear los puertos.  Podemos hacer la redirección de puertos con las opciones:
    -p: especificamos a qué puerto queremos la redirección.
    -P: le dice a Docker que si expone algún tipo de puerto haga el forwarding a un puerto aleatorio. (Dockerfile)

    $ docker run -d -p 8000:80 --name apacheServer2 apache2/ubuntu:v1 /usr/sbin/apache2ctl -D FOREGROUND
    $ docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES  
        a5e0a6d83f61        apache2/ubuntu:v1   "/usr/sbin/apache2ct…"   5 seconds ago       Up 4 seconds        0.0.0.0:8000->80/tcp   apacheServer2

 Ahora si probamos en el navegador la url http://localhost:8000  se mostrará la página web de inicio de Apache2.

# Links
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

# Volúmenes
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

# Configuración de RED.        
 Para ver los tipos de red disponibles por defecto en Docker, sólo hay que ejecutar:
        
        $ docker network ls
            NETWORK ID          NAME                DRIVER
            eb2291dd0b63        bridge              bridge              
            b9e68dd550d4        host                host                
            ccc5be97afaf        none                null               

    *· host* Representa la red del propio equipo y haría referencia a eth0
    *· bridge* representa la red docker0 y a ella se conectan todos los contenedores por defecto.
    *· none* siginifica que el contenedor no se incluye en ninguna red y si verificamos esto con el comando ifconfig dentro del contenedor veríamos que solo tiene interfade de loopback lo.

    Se pueden ver o inspecionar los detalles de cada red usando:

        $ docker network inspect bridge

    ** BRIDGE **
        Puede albergar a 65534 contenedores. Tras ejecutar el comando anterior también podremos ver si esta red tiene o no tiene contenedores en funcionamiento en la red. Todos los contenedores dentro de esta red pueden verse los unos a los otros.

    ** HOST **

    ** NONE **




