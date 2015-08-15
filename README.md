# Imagen docker de Kimkelen 

Simplifica la instalación de kimkelen

## Instalar docker

En Ubuntu, la instalación es:

```
curl -sSL https://get.docker.com/ | sh
```

Para otros sistemas verificar la [Guía de instalación de
Docker](https://docs.docker.com/installation/)

## Probando kimkelen

Si no se quiere leer todo este README y simplemente probar kimkelen rápidamente:

###Iniciamos un contenedor con MySQL:

```
docker run --name=mysql-kimkelen -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=kimkelen -e MYSQL_USER=kimkelen \
  -e MYSQL_PASSWORD=kimpass -d mysql:5.5
```

### Iniciamos un contenedor con kimkelen ligado al MySQL creado

```
docker run -e USER_ID=`id -u` -e DB_NAME=kimkelen -e DB_USER=kimkelen \
  -e DB_PASS=kimpass -e DB_HOST=mysql -v /tmp/kimkelen/codigo:/code \
  -v /tmp/kimkelen/data:/data --name=kimkelen --link mysql-kimkelen:mysql  \
  -p 8000:80  -it cespi/kimkelen
```

Luego accceder http://localhost:8000

## Instalar el contenedor de kimkelen

Correr por única vez:

```
docker run \
    -e USER_ID=`id -u` \
    -e DB_NAME=kimkelen \
    -e DB_USER=kimkelen \
    -e DB_PASS=muysecreta \
    -v /tmp/kimkelen/codigo:/code \
    -v /tmp/kimkelen/data:/data \
    --name=kimkelen \
    -p 8000:80 \
    -it
    cespi/kimkelen
```

Esto creará el contenedor que instala la última versión de kimkelen. Las opcones
del comando anterior especifican:

* *USER_ID:* el usuario con el que correrá la aplicación. En el ejemplo coincide
  con el usuario que ejecuta el comando
* *DB_XXX:* datos de conexión a la base de datos. La primera vez que se ejecura
  el comando, configura la DB con estos datos
* *Mapeo de volúmenes:* los datos del contenedor se manejan en dos volumenes
  docker:
  * `/code`: mantiene las distintas versiones de kimekelen
  * `/data`: mantiene los datos persistentes comunes a las diferentes versiones
    de kimkelen instaladas: uploads de archivos, configuraciones, logs, etc
* *Nombre del contenedor:* el contenedor en este caso se llamará kimkelen y nos
  permitirá referenciarlo fácilmente en otros comandos docker
* *Mapeo de puertos:* el contenedor escuchará en el puerto 8000, mapeando este
  puerto al 80 del contenedor. Esto significa que kimkelen quedará accesible
  directamente en el puerto 8000 de la máquina donde se corre docker
* *Correr en modo intereactivo:* las opciones `-it` corren el contenedor en modo
  interactivo y así responder a la única interacción del instalador: si se desea
  reinicializar la base de datos.

Una vez completado el comando anterior, quedará funcionando kimkelen en el
puerto 8000.

### Reinstalando kimkelen

Si se desea reinstalar la versión de kimklen y/o reconfigurarlo, es posible
correr el contenedor con el comando  siguiente:

```
docker run \
    -e USER_ID=`id -u` \
    -e DB_NAME=kimkelen \
    -e DB_USER=kimkelen \
    -e DB_PASS=muysecreta \
    -v /tmp/kimkelen/codigo:/code \
    -v /tmp/kimkelen/data:/data \
    -p 8000:80 \
    -it \
    cespi/kimkelen --reinstall
``` 

Este comando lo que hace es:

* Reinstala la versión específica de kimkelen (la indicada por GIT_REVISION)
* Recrea los archivos:
  * `config/propel.ini`
  * `config/databases.yml`
  * `config/nc_flavor.yml`

Notar que no utilizamos `--name` y utilizamos `--rm`. Esto lo que hace es crear
un nuevo contenedor que solo actualizará los mismos volumenes que utiliza el
contenedor kimkelen. 

Al utilizar `--reinstall` *no se inicia el web server*

## Usar mysql en un contenedor

Podemos correr mysql en un contenedor de la siguiente forma 

```
docker run \
  --name=mysql-kimkelen \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=kimkelen \
  -e MYSQL_USER=kimkelen \
  -e MYSQL_PASSWORD=kimpass \
  -d mysql:5.5 
```

Una vez corriendo, debemos iniciar nuestra instancia de kimkelen de esta forma:

```
docker run \
    -e USER_ID=`id -u` \
    -e DB_NAME=kimkelen \
    -e DB_USER=kimkelen \
    -e DB_PASS=kimpass \
    -e DB_HOST=mysql \
    -v /tmp/kimkelen/codigo:/code \
    -v /tmp/kimkelen/data:/data \
    --name=kimkelen \
    --link mysql-kimkelen:mysql \
    -p 8000:80 \
    -it \
    cespi/kimkelen
```


*Si ya estaba existe un contenedor docker con el nombre kimkelen, deberá antes
pararlo y eliminarlo: `docker stop kimkelen && docker rm kimkelen`*


## Iniciar y parar el contenedor

Una vez creado el contenedor como se explica en el punto anterior, se está
creando un contenedor *nombrado kimkelen*. Este nombre podemos usarlo para:

* *Parar el conentedor:* `docker stop kimkelen`
* *Iniciar el conentedor:* `docker start kimkelen`
* *Ver los logs:* `docker logs -f kimkelen`

## La conexión a la base de datos

Dado que el contenedor corre en un segmento de red diferente, la base de datos
deberá admitir conexiones desde la red. Es por ello que si existen problemas de
conexión  a la base de datos, deberá verificar que la configuración sea la adecuada:

* Que los datos de nombre de base de datos, host, usuario y contraseña sean los
  esperados
  * Los archivos de configuración pueden verse en el volumen montado por docker
    bajo `data/config/`
* Que se disponga de permisos para conectarse desde la red de docker
  * Si el problema es de permisos accediendo desde otro host, verificar que:
    * El servidor de mysql esté escuchando en un puerto válido (no en 127.0.0.1)
    * Que el usuario tenga permisos de acceso: 

```
GRANT ALL PRIVILEGES ON kimkelen_docker.* to kimkelen@'%' identidied by 'muysecreta';`
```


## Los volumenes

El contenedor de kimkelen provee dos volumenes:

* `code/`: donde se almacenan las diferentes versiones de kimkelen que se vayan
  actualizando. El directorio contendrá las distintas versiones que vayan
descargándose. Ver *GIT_REVISION*. 
* `data/`: configuraciones y datos persisntentes entre las diferentes
  versiones instaladas. Aquí se mantienen:
  * `data/php.ini`: configuración de PHP
  * `data/config/databases.yml`: configuración de la base de datos
  * `data/config/propel.ini`: configuración de la base de datos
  * `data/config/app.yml`: configuración de la aplicación básica
  * `data/data/`: datos donde se almacenan documentos subidos al sistema
  * `data/log/`: logs de la aplicación symfony (no del web server)
  * `data/web/uploads`: otros uploads de la aplicación

Un volumen en docker, permite peristir los datos del contenedor. Para más
información ver [la sección de Volúmenes del
manual](https://docs.docker.com/userguide/dockervolumes/)

La aplicación podrá correrse con un usuario estándar si se especifica la
variable de ambiente *USER_ID*

## Interacción con el contenedor

Este contenedor permite correr comandos symfony dentro del contenedor de la
siguiente forma:

```
docker run \
    -e USER_ID=`id -u` \
    -v /tmp/kimkelen/codigo:/code \
    -v /tmp/kimkelen/data:/data \
    -it \
    --rm \
    cespi/kimkelen \
    symfony #comando#
```

Si se desea acceder al contenedor utilizando un shell bash como root por ejemplo:

```
docker run \
    -e USER_ID=`id -u` \
    -v /tmp/kimkelen/codigo:/code \
    -v /tmp/kimkelen/data:/data \
    -it \
    --rm \
    --entrypoint=bash \
    cespi/kimkelen
```

## Opciones al correr el contenedor

El contenedor mínimamente deberá definir el puerto en el que correrá

### Definir el puerto

Se exponen los puertos 80 y 443. La forma de asociar estos puertos con puertos
de la máquina local es con la opción `-d PTO_LOCAL:PTO_DESTINO`, por ejemplo `-d
8000:80` indica que al acceder al puerto 8000 de la pc donde se corre el
contenedor redireccionará a kimkelen

### Definir los volúmnes

Pueden utilizarse volúmenes que maneja docker o directorios de la PC donde se
correrá docker. Si se utiliza la opción:

```
    -v `pwd`/code:/code -v `pwd`/data:/data
```

Estamos asociando los directorios `/code` y `/data` del contenedor con dos
directorios de la PC local ubicados en el directorio actual (por ello pwd) con
los mismos nombres respectivamente

### Especificar el usuario USER_ID

Si corremos el contenedor sin especificar un usuario, todas las operaciones se
realizan con el usuario www-data que en Debian es el UID 33. Esto nos complicará
la interacción con las configuraciones alojadas en los volúmenes dado que necesitaremos
privilegios de root para modificar estos archivos.
Para simplificar esta cuestión, se provee la posibilidad de indicar con qué UID
deberá correr kimkelen. Esto es posible con la opción `-e USER_ID=1000` o por
ejemplo

```
    -e USER_ID=`id -u`
```

Que seteará esta variable al UID del usuario actual

### Datos de conexión a la base de datos

Pueden definirse los siguientes valores: DB_NAME, DB_HOST, DB_USER, DB_PASS

Las variables de entorno mencionadas se pasan al contenedor de la siguiente
forma:

```
    -e DB_NAME=kimkelen_prueba
```

*Es importante destacar que una vez creados los archivos `config/databases.yml`
y `config/propel.ini` estas variables serán ignoradas. Si se desea recrear estos
archivos deberá eliminarlos y volver a correr el contenedor*

### Definir la versión de kimkelen: GIT_REVISION

El contenedor instalará la versión master, pero puede cambiarse este hecho
definiendo la variable *GIT_REVISION* de la siguiente forma:

```
    -e GIT_REVISION=v2.19.4
```

## Actualizando el contenedor

Para obtener la última versión de esta imagen, correr: 

```
docker pull cespi/kimkelen
```
