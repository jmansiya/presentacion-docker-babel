---
layout: default
title: DOCKER-COMPOSE. LISTA DE KEYS SOPORTADAS.
---
# {{page.title}}

A continuación se muestra una lista de keys soportadas en el docker-compose:



- **image**: Es la ‘tag’ o id de la imagen.
- **build**: Es el ‘path’ al directorio que contiene un ‘Dockerfile’
- **command**: Es la key que reemplaza el comando por defecto.
- **links**: Es la key que enlaza  a contenedores con otro servicio.
- **external_links**: Esta key enlaza a contenedores iniciados por otro docker-compose.yml o por algún otro medio.
- **ports**: Esta key expone a los puertos y específica los puertos HOST_port:CONTAINER_port
- **expose**: Esta key expone los puertos sin publicarlos en el host.
- **volumes**: Esta key monta ‘paths’ como volúmenes.
- **volumes_from**: Esta key monta todos los volúmenes desde otro contenedor.
- **environment**: Añade las variables de entorno
- **env_file**: Añade las variables de entorno a un archivo
- **extends**: Extiende de otro servicio definido en el mismo u otro archivo de configuración
- **net**: Es el modo de red, el cual tiene el mismo valor que el Docker client –-net option
- **pid**: Permite el intercambio de espacio PID entre el host y los contenedores
- **dns**: Establecemos servidores DNS personalizados
- **cap_add**: Añade una capa al contenedor.
- **cap_drop**: Elimina una capa al contenedor
- **dns_search**: Establece servidores de búsqueda DNS personalizados
- **working_dir**: Cambia el directorio de trabajo dentro del contenedor.
- **entrypoint**: Esto reemplaza el punto de entrada predeterminado.
- **user**: Establece el usuario predeterminado
- **hostname**: Establece un nombre de host del contenedor
- **domainname**: Establece un nombre de dominio
- **mem_limit**: Limita la memoria
- **privileged**: Proporciona privilegios ampliados
- **restart**: Establece la política de reinicio del contenedor.
- **stdin_open**: Permite el dispositivo de entrada estándar.
- **tty**: Permite el control basado en texto, como un terminal
- **cpu_shares**: Establece las partes de la CPU.