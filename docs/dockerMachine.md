---

layout: default
title: Demo Docker-machine
show_menu: true
weight: 0
title_menu: Docker-machine
---

# {{page.title}}

En esta demo veremos la ejecución de algunos comandos de docker-machine (Dependiendo del SO que tengamos necesitaremos instalar una utilizando Virtual Box, para poder hacer funcionar en nuestros equipos docker):

## Crear una nueva máquina de docker.

    $ docker-machine create --driver virtualbox pruebas   

Si tenemos windows 10 con hyperv el driver sería: 'hyperv'
    
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