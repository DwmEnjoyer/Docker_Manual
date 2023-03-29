![Screenshot](docker.svg)
# ***Docker Manual***
## Docker para nodos individuales
### Contenedores
Ejecutar un contenedor:
```bash
docker run <nombreimagen>
o
docker container run <nombreimagen>
##Opciones
-d -> Ejecuta el contenedor en seundo plano
-it -> Asocia una shell para interactuar con el contenedor
-itd ->Una mezcla de las dos opciones anteriores
-p <phost>:<pcontenedor> -> Abre puertos
--rm -> Elimina el contenedor cuando se detiene
--name -> Ponerle un nombre al contenedor
--network <red>(bridge/host/none/redpersonalizada) -> Asocia una red, por defecto es bridge
--volume <volumen>:<mountpoint> o <rutahost>:<mountpoint> -> Asocia un volumen para datos persistentes o monta una ruta del host en el contenedor, si la ruta no existe la crea
--mount type=<tipo>(bind/tmpfs),source=<rutahost>,target=<mountpoint> -> Monta un directorio del host en el contenedor, si la ruta no existe no crea nada. Se puede usar para tmpfs, en cuyo caso no se especifica la fuente, sólo el destino
--tmpfs -> Emplea memoria volátil del host para guardar información no persistente, es decir, desaparece al reiniciar el contenedor(esta opción no está disponible en Docker Swarm)
```
Listar los contenedores en ejecución:
```bash
docker ps
##Opciones típicas
-a ->Para listar todos los contenedores
-s ->Para comprobar cuánto espacio ocupan los contenedores en ejecución
```
Entrar con una shell nueva en el contenedor:
```bash
docker exec -it <nombrecontenedor> <shell>
```
Entrar con la shell original en el contenedor:
```bash
docker attach <nombrecontenedor>
```
### Imágenes
Para listar las imágenes disponibles en local:
```bash
docker image ls
```
Para listar las imágenes pertenecientes a un repositorio concreto:
```bash
docker image ls <nombreimagen>
```
Para eliminar las imágenes poco usadas(dangling):
```bash
docker image prune
```
Para eliminar las imágenes sin contenedores asociados:
```bash
docker image prune --all
```
Mostrar información sobre una imagen determinada:
```bash
docker image inspect <nombreimagen>
```
Descargar una imagen:
```bash
docker pull <nombreimagen>:<tag>
```
Buscar una imagen:
```bash
docker search <nombreimagen>
```
Contruir una imagen:
```bash
docker build -t <nombreimagen>:<tag> <buildContext>(normalmente ., es decir, el directorio actual)
```
Consultar capas(historial) de una imagen:
```bash
docker image history <nombreimagen>:<tag>
```
Construir imagen a partir de un contenedor:
```bash
docker container commit <nombreContenedor> <nombreImagen>:<tag>
##Opciones típicas
--message "mensaje"
--author "autor"
```
### Networking
Listar las redes disponibles(por defecto tres, bridge, host y none):
```bash
docker network ls
```
Mostar información sobre una red determinada:
```bash
docker network inspect <nombrered>
```
Añadir red a un contenedor:
```bash
docker network connect <nombrered>
```
Eliminar red de un contenedor:
```bash
docker network disconnect <nombrered>
```
Crear una red:
```bash
docker network create <nombrered>
##Opciones
--driver=<nombre>(bridge/overlay) -> Driver de la red, por defecto bridge. Emplearemos overlay si deseamos crear una red para un clúster
--subnet=<ipred>/<bitsred> -> Personalizar ip de la red
--gateway=<ippuertaenlace> -> Especificar ip de la puerta de enlace por defecto de la red
```
Eliminar una red. Si la red está siendo utilizada por un contenedor tendremos que pararlo antes de eliminarla:
```bash
docker network rm <nombrered>
```
Para eliminar todas las redes que no están siendo utilizadas por ningún contenedor:
```bash
docker network prune
```
### Almacenamiento
Información sobre almacenamiento a nivel general:
```bash
docker system df
##Opciones
-v -> Información más detallada
```
Para cambiar de driver :
```bash
dockerd --storage-driver <driveralmacenamiento>
```
Crear volumen de almacenamiento(ruta en host es ***/var/lib/docker/volumes/nombre/_data***):
```bash
docker volume create <nombrevolumen>
```
Listar volúmenes de almacenamiento:
```bash
docker volume ls
```
Eliminar volúmenes:
```bash
docker volume rm <nombrevolumen>
```
Información sobre el volumen:
```bash
docker volume inspect <nombrevolumen>
```
### Dockerfile
Definir argumentos de entrada:
```bash
ARG <NOMBRE>=<valor>
```
Definir imagen base:
```bash
FROM <imagen>:${ARGUMENTO}
```
Ejecutar comandos:
```bash
RUN <comando>
o
RUN ["programa", "parámetro"]
```
Ejecutar comando tras lanzar el contenedor(sólo se puede usar una vez por dockerfile):
```bash
CMD <comando> <parámetro>
o
CMD ["programa", "parámetro"]
```
Permite especificar el ejecutable por defecto del contenedor, si se utiliza en conjunto con `CMD` no necesitaremos poner qué ejecutar, sino los parámetros a utilizar(En `ENTRYPOINT` ponemos el ejecutable y parámetros por defecto en `CMD` los parámetros extra):
```bash
ENTRYPOINT <comando> <parámetro>
o
ENTRYPOINT ["programa", "parámetro"]
```
Para establecer variables de entorno:
```bash
ENV=<valor>
```
Establecer directorio en el que se ejcutarán los distintos comandos:
```bash
WORKDIR /ruta/de/ejemplo
```
Añade archivos del entorno de build a la imagen:
```bash
COPY <fuente> <destino>(Ruta dentro o fuera del WORKDIR)
```
Copia archivos o directorios del entorno de build  a la imagen, pero tiene funcionalidades extra:
```bash
ADD <fuente> <destino>
```
### Docker Compose
Docker compose nos permite levantar aplicaciones que requieren más de un contenedor para su funcionamiento. Esto se consigue a través de la creación de un sencillo archivo llamado `docker-compose.yaml` o `docker-compose.yml` en el que declararemos las imágenes a instanciar, sus variables de entorno, las redes a utilizar... Tras ello, para poner en marcha los contenedores ejecutaremos:
```bash
docker compose up
o
docker-compose up
```
Y para detenerlos:
```bash
docker compose down
o
docker-compose down
```
## Docker para clusters
### Docker Swarm
#### Nodos
Instanciar el swarm:
```bash
docker swarm init 
##Opciones
--advertise-addr <ip> -> Ip del manager accesible desde el resto de nodos para unirse al swarm
```
Generar clave para permitir el acceso a nuevos nodos:
```bash
docker swarm join-token <tiponodo>(manager/worker)
##Opcion
--rotate -> Para generar nueva clave
```
Meter un nuevo nodo en el swarm:
```bash
docker swarm join <advertiseaddr>
##Opciones
--token <token> -> Token generado por un manager del swarm al que deseamos unirnos
```
Para listar los nodos del swarm(desde un nodo manager):
```bash
docker node ls
```
Mostrar información sobre un nodo determinado:
```bash
docker node inspect <nombrenodo>
##Opciones
--prettier -> Para que sea más legible
```
Modificar características de un nodo determinado:
```bash
docker node update <nombrenodo>
##Opciones
--label-add <key o key/value> -> Añadir etiquetas a los nodos
--availability <estado>(drain/active) -> Privar a un nodo de realizar tareas, si tiene tareas en ejecución las detiene
```
Ascender nodo a manager:
```bash
docker node promote <nombrenodo>
```
Descender nodo a worker:
```bash
docker node demote <nombrenodo>
```
Sacar nodo actual del swarm:
```bash
docker swarm leave
```
Sacar un nodo determinado desde un nodo manager:
```bash
docker node rm <nombrenodo>
```
#### Servicios
Crear un servicio:
```bash
docker service create <nombreservicio>
##Opciones
--name -> Asignar un nombre a los contenedores
--replicas <numero> -> Número de contenedores a desplegar del servicio designado
--mount type=<tipo>(bind/tmpfs/volume),source=<rutahost>,target=<mountpoint> -> Similar al mount en un nodo, es este caso se hace en todos los nodos
--update-delay <tiempo>(segundos, minutos u horas) -> Tiempo de espera entre actualizaciones de los distintos nodos2
--mode global <tarea1> <tarea2> <tarea3> -> Crear un servicio formado por tareas distintas distribuidas en los distintos nodos
--network <redoverlay> -> Permite añadir una red overlay personalizada
--publish published=<hostport>,target=<containerport>,protocol=<udp/tcp>(por defecto tcp),mode=<modo>(host) -> Abrir y mapear puertos de los distintos nodos. Si empleamos el mode=host aseguramos que al acceder a uno de los nodos estamos efectivamente accediendo al contenedor de ese nodo(por defecto puede pasar que al acceder acabemos en otro nodo)
--reserve-memory <memoria> -> Para establecer el mínimo de memoria a emplear por los nodos en los que se ejecute el servicio
--reserve-cpu <cpu> -> Para establecer el mínimo de cores de CPU a emplear por los nodos en los que se ejecute el servicio
--constraint node.labels.region==<region> -> Especificar qué nodos pueden ejecutar las tareas de nuestro servicio, en este caso dependiendo de la región del nodo
--hostname=<hostname> -> Nos permite establecer el nombre de los contenedores que ejecutan nuestra tarea empleando texto plano o usando sintaxis de Golang para que el nombre de cada contenedor varíe, por ejemplo, dependiendo del nombre del nodo en el que se esté ejecutando
--secret <nombresecreto> -> Especificamos la información protegida a emplear por el clúster
```
Listar servicios:
```bash
docker service ls
```
Información sobre un determinado servicio:
```bash
docker service inspect <nombreservicio>
##Opciones
--pretty -> Información más entendible
```
Consultar las tareas que componen un determinado servicio(ver los contenedores desplegado en los distintos nodos):
```bash
docker service ps <nombreservicio>
```
Escalar o reducir un servicio:
```bash
docker service scale <nombreservicio>=<numero>
```
Eliminar un servicio(y  todas las tareas asociadas):
```bash
docker service rm <nombreservicio>
```
Modificar las características de un servicio(añadir o eliminar volúmenes, por ejemplo):
```bash
docker service update <nombreservicio>
##Opciones
--mount-add type=<tipo>(bind/tmpfs/volume),source=<rutahost>,target=<mountpoint> -> Para añadir un nuevo volumen a un servicio en funcionamiento(Se detienen los contenedores en ejecución y se generan unos nuevos con las características especificadas)
--mount-rm <mountpoint> -> Para eliminar volúmenes de almacenamiento
--image <nombreimagen> -> Utiliza una nueva imagen e instancia nuevos contenedores, respetando la política de actualización
```
#### Manejo de secretos
Con los siguientes comandos podremos manejar información sensible a través del clúster, creando para ello secretos que contienen la información crítica a proteger:
Para crear un secreto:
```bash
docker secret create <nombresecreto> <archivosecreto>
```
Para eliminarlo:
```bash
docker secret rm <nombresecreto>
```
Para listar los distintos secretos:
```bash
docker secret ls
```
#### Docker Stack
Similar a Docker Compose pero para clusters, nos permite organizar y desplegar  una aplicación compuesta por diversos servicios a través de los nodos del clúster. De igual manera que Docker Compose se emplea un archivo `docker-compose.yaml` y lo levantamos de la siguiente manera:
```bash
docker stack deploy -c <nombrecompose> <nombrestack>
```
Para listar los stacks:
```bash
docker stack ls
```
### Administración del cluster
#### Universal Control Plane
Con UCP podemos administrar y controlar el acceso a los recursos presentes en los nodos de nuestro cluster, ya sea Docker Swarm o Kubernetes. También nos permite integrar servicios de autenticación que tengamos desplegados en otro servidor, como podría ser LDAP. Podemos interactuar con UCP a través de la línea de comandos o desde la interfaz web. Para ponerlo en funcionamiento simplemente hacemos pull de la imagen y la ponemos en funcionamiento en un nodo de confianza:
```bash
docker run -it -v /var/run/docker.sock:/var/run/docker.sock docker/ucp 
```
Como podemos observar, hemos de darle acceso al socket de docker para que UCP pueda comunicarse correctamente con el servicio. 
#### Docker Trusted Registry
Con DTR podemos almacenar imágenes propias en nuestro propio servidor y requiere el uso de UCP para control de acceso. Lo podemos poner en funcionamiento de la siguiente manera:
```bash
docker run -it docker/dtr --ucp-node <nombrenodo> --ucp-username <nombreadmin> --ucp-url <url> --ucp-insecure-tls(si usamos certificados autofirmados o algo por el estilo) 
```
Para poder hacer pull o push de nuestro nuevo registro hemos de logearnos primero:
```bash
docker login <ipnododtr>
```
Si deseamos mantener varias copias de DTR en nuestro clúster para aumentar la disponibilidad podemos ejecutar el siguiente comando en otros nodos:
```bash
docker run -it docker/dtr join --ucp-node <nombrenodo> --ucp-username <nombreadmin> --ucp-url <url> --ucp-insecure-tls
```
