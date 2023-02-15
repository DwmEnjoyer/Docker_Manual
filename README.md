# ***Docker Bible***
## Comandos básicos
Ejecutar un contenedor:
```bash
docker run <nombreimagen>
##Opciones
-d -> Ejecuta el contenedor en seundo plano
-it -> Asocia una shell para interactuar con el contenedor
-itd ->Una mezcla de las dos opciones anteriores
-p <phost>:<pcontenedor> -> Abre puertos
--name -> Ponerle un nombre al contenedor
--network <red> -> Asocia una red, por defecto es bridge
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
## Networking
## Imágenes
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
## Dockerfile
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
## Almacenamiento
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
## Swarm
### Administración de nodos
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
Añadir etiquetas a los nodos:
```bash
docker node update <nombrenodo>
##Opciones
--label-add <key o key/value> -> Añadir etiquetas a los nodos
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
### Administración de servicios
Crear un servicio:
```bash
docker service create <nombreservicio>
##Opciones
--name -> Asignar un nombre a los contenedores
--replicas <numero> -> Número de contenedores a desplegar del servicio designado
--mount type=<tipo>(bind/tmpfs/volume),source=<rutahost>,target=<mountpoint> -> Similar al mount en un nodo, es este caso se hace en todos los nodos
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
### Actualización de nodos
```bash
docker service update <nombreservicio>
##Opciones
--mount-add type=<tipo>(bind/tmpfs/volume),source=<rutahost>,target=<mountpoint> -> Para añadir un nuevo volumen a un servicio en funcionamiento(Se detienen los contenedores en ejecución y se generan unos nuevos con las características especificadas)
```
