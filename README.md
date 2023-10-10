# Tarea 3

Haz un fork de este repo

NOTA: Esta tarea se puede ejecutar en el playground de docker.

https://labs.play-with-docker.com/

## Actividad 1: Mi primer docker-compose

Revisa el archivo `docker-compose.yml`.

Revisa el archivo `.env-sample`

El archivo `.env-sample` define las variables de entorno usadas en el archivo `docker-compose.yml`.

Copia el archivo `.env-sample` a `.env`, en linux esto se hace así:

        cp .env-sample .env

Edita el archivo `.env`. Cambia los valores de las variables que quieras.
Graba el archiv `.env`. (En linux puedes usar el editor `nano` o `vim`).

Ejecuta docker-compose de este modo:

        docker-compose --env-file .env-sample up -d

Ahora revisa cuantos contenedores están corriendo de este modo:

        docker ps

Ingresa a la base de datos con este comando (reemplaza los valores que corresponde a tu configuración en el archivo `.env`):

        docker-compose exec -it postgres psql -U POSTGRES_USER POSTGRES_DB

Por ejemplo, usando los valores que están en el archiv `.env-sample`, debes hacer:


        docker-compose exec -it postgres psql -U postgres usuarios


Con esto puedes ejecutar comandos en psql para revisar la base de datos:

        psql (15.4)
        Type "help" for help.

        usuarios=# select * from users;


Para salir escribe `\q`.


Deten tus contenedores con este comando:

        docker-compose down

## Actividad 2: la aplicación chatroom

Edita el archivo `docker-compose.yml` y agrega los servicios de frontend y backend:

```
  frontend:
    build: chat-frontend
    restart: always
    environment:
      - FRONT_PORT
    expose: 
      - 80
    ports:
      - ${FRONT_PORT}:80
    volumes:
      - ./nginx-conf/default.conf:/etc/nginx/conf.d/default.conf
 
  users-svc:
    build: users-svc
    command: "node app.js" 
    restart: always
    expose:
      - 3000
    environment:
      - JWT_SECRET
      - POSTGRES_USER
      - POSTGRES_PASSWORD
      - POSTGRES_DB
      - POSTGRES_PORT=5432
      - POSTGRES_SERVER=postgres
      - PORT=3000
      - CONNECTION_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
    depends_on:
      - postgres
  
  ws-server:
    build: ws-server
    command: "node index.js"
    restart: always
    expose:
      - 3001
    environment:
      - PORT=3001
```

Asegurate de colocar un valor adecuado a la variable `FRONT_PORT`` en tu archivo `.env`

Levanta los contenedores con el comando:

        docker-compose up -d

Navega con browser a la dirección `localhost:FRONT_PORT`, deberias ver la pagina de login.

Registra un usuario, la aplicación debería estar corriendo correctamente.

# Nombre
Felipe Zambrano

## Preguntas

1. Revisa el archivo `Dockerfile` en la carpeta `users-svc` y compáralo con el mismo archivo en la carpeta `ws-server`. ¿Qué te llama la atención? Ahora revisa la sentencia `command` para los respectivos servicios en el archivo `docker-compose.yml`. ¿Qué concluyes?
R. Los dos archivos Dockerfile son idénticos, por lo que no hay ninguna diferencia notable entre ellos. Ambos utilizan la misma imagen base de Docker (node:18-alpine), establecen un directorio de trabajo en /usr/app, copian el archivo package.json al directorio de trabajo, ejecutan npm install --quiet para instalar las dependencias del proyecto, y finalmente copian todo el contenido del contexto de construcción (incluidos los archivos locales) al directorio de trabajo en el contenedor Docker.

En conclusión:
* Los servicios frontend y backend utilizan los Dockerfiles indicados anteriormente y ejecutan los comandos predeterminados del contenedor Node.js para iniciar las aplicaciones.
* Los servicios ws-server y flyway también utilizan los Dockerfiles indicados y ejecutan comandos personalizados específicos para iniciar sus respectivas aplicaciones.


2. Revisa el archivo `Dockerfile` en la carpeta `frontend`. ¿Qué te llama la atención? ¿En qué es diferente de los otros archivos `Dockerfile`?
R. En comparación con los otros Dockerfiles, este utiliza la construcción en dos etapas para optimizar el tamaño del contenedor final. La etapa de construcción incluye todas las dependencias de desarrollo y realiza la compilación, mientras que la etapa de liberación solo incluye los archivos necesarios para ejecutar la aplicación en un servidor Nginx. Los Dockerfiles anteriores no utilizan una construcción en dos etapas y simplemente instalan las dependencias y copian los archivos al contenedor, sin dividir el proceso en etapas de construcción y liberación.

3. ¿Para qué sirve el servicio flyway? ¿Qué pasa al hacer `docker ps` con respecto a este servicio?
R. Flyway es una herramienta de migración de bases de datos que permite a los desarrolladores manejar las versiones de las bases de datos de manera estructurada y automática. Facilita la gestión de esquemas de bases de datos y versiones, lo que es crucial en aplicaciones en desarrollo continuo o en colaboración con múltiples desarrolladores. Cuando se ejecuta docker ps, se obtiene una lista de los contenedores Docker que están actualmente en ejecución en el sistema. Esto incluye los contenedores que fueron lanzados con docker run y están actualmente en ejecución. Sin embargo, docker ps no muestra los contenedores que han finalizado su ejecución.

4. ¿Cuantas imágenes se crean? ¿Cuántos contenedores están activos?
R. En el archivo docker-compose.yml, se definen cuatro servicios: frontend, backend, ws-server, y flyway. Cada uno de estos servicios se basa en una imagen de Docker o se construye a partir de un Dockerfile. Asumiendo que todas las imágenes definidas en el archivo docker-compose.yml se construyen o descargan correctamente, se crearán cuatro imágenes de Docker. Hay 2 contenedores activos.

5. Deten los contenedores con `docker-compose down`, luego reinicia con `docker-compose up -d`. Ingresa a la base de datos. ¿Qué pasa con los datos? 
R. Después de reiniciar, se puede acceder a los datos de la base de datos.

6. Baja los contenedres. Crea un volumen para postgres agregando estas sentencias en el servicio `postgres`: 

```
 volumes:
   - ./data:/var/lib/postgresql/data
```

Reinicia los contenedores. Explica qué pasa con la base de datos después de hacer esto.

¿Qué pasa con la carpeta `data`, qué crees que contiene?
Cuando se reinician los contenedores después de agregar esta configuración de volumen y después de que se ha creado la base de datos en el contenedor PostgreSQL, los datos de la base de datos se almacenan en la carpeta ./data en la máquina local. Esto significa que incluso si se detiene o se elimina el contenedor de PostgreSQL, los datos de la base de datos se mantendrán en la carpeta ./data.

