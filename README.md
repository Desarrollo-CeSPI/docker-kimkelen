# Imagen docker de Kimkelen 

Simplifica la instalación del sistema
[Kimkelen](git@github.com:Desarrollo-CeSPI/docker-kimkelen.git).

## Instalar docker

En Ubuntu ejecutar:

```
curl -sSL https://get.docker.com/ | sh
```

Para otros sistemas operativos verificar la [Guía de instalación de
Docker](https://docs.docker.com/installation/).

# Guía de inicio rápido

Si no quiere leer todo este README y simplemente desea probar Kimkelen rápidamente:

## Iniciar un contenedor con MySQL:

```
docker run --name=mysql-kimkelen -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=kimkelen -e MYSQL_USER=kimkelen \
  -e MYSQL_PASSWORD=kimpass -d mysql:5.5
```

## Iniciar un contenedor con Kimkelen ligado al MySQL creado

```
docker run -e USER_ID=`id -u` -e DB_NAME=kimkelen -e DB_USER=kimkelen \
  -e DB_PASS=kimpass -e DB_HOST=mysql -v /tmp/kimkelen/codigo:/code \
  -v /tmp/kimkelen/data:/data --name=kimkelen --link mysql-kimkelen:mysql \
  -p 8000:80 -it cespi/kimkelen
```

## Acceder al sistema

El mismo estará disponible ingresando a la URL: http://localhost:8000. Para
información acerca de los datos precargados (incluyendo usuario y contraseña de
acceso) referirse a la sección _Datos iniciales_ en el [README de
Kimkelen](https://github.com/Desarrollo-CeSPI/kimkelen#datos-iniciales-1).

# Guía detallada

A continuación se explica en detalle lo que se mostró anteriormente. Si se tiene
corriendo un contenedor llamado *kimkelen* el siguiente comando dará error
porque ya se creó una instancia con ese nombre. Deberá pararla y eliminarla
antes de seguir adelante.

Para iniciar un contenedor con Kimkelen se utiliza el comando:

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
    -it \
    cespi/kimkelen
```

Esto creará el contenedor que instala la última versión de Kimkelen. Las opciones
del comando anterior especifican:

* *USER_ID:* el usuario con el que correrá la aplicación. En el ejemplo coincide
  con el usuario que ejecuta el comando.
* *DB_XXX:* datos de conexión a la base de datos. La primera vez que se ejecuta
  el comando, configura la DB con estos datos.
* *Mapeo de volúmenes:* los datos del contenedor se manejan en dos volúmenes
  Docker:
  * `/code`: mantiene las distintas versiones de Kimekelen.
  * `/data`: mantiene los datos persistentes comunes a las diferentes versiones
    de Kimkelen instaladas: uploads de archivos, configuraciones, logs, etc.
* *Nombre del contenedor:* el contenedor en este caso se llamará kimkelen y nos
  permitirá referenciarlo fácilmente en otros comandos Docker.
* *Mapeo de puertos:* el contenedor escuchará en el puerto 8000, mapeando este
  puerto al 80 del contenedor. Esto significa que Kimkelen quedará accesible
  directamente en el puerto 8000 de la máquina donde se corre Docker.
* *Correr en modo interactivo o detached:* las opciones `-it` corren el contenedor en modo
  interactivo y permiten así cancelar la ejecución del contenedor con utilizando
  `Ctrl+C`. Si se desea evitar este modo, omitir las opciones `-it` y utilizar
  `-d`

Una vez completado el comando anterior, quedará funcionando Kimkelen en el
puerto 8000.


## Usar MySQL en un contenedor

Podemos correr MySQL en un contenedor de la siguiente forma:

```
docker run \
  --name=mysql-kimkelen \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=kimkelen \
  -e MYSQL_USER=kimkelen \
  -e MYSQL_PASSWORD=kimpass \
  -d mysql:5.5 
```

Una vez corriendo, debemos iniciar nuestra instancia de Kimkelen de esta forma:

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

*Si ya existía un contenedor Docker con el nombre kimkelen, deberá antes
pararlo y eliminarlo: `docker stop kimkelen && docker rm kimkelen`*


## Iniciar y parar el contenedor

Una vez creado el contenedor como se explica en el punto anterior, se está
creando un contenedor *llamado kimkelen*. Este nombre podemos usarlo para:

* *Parar el conentedor:* `docker stop kimkelen`.
* *Iniciar el conentedor:* `docker start kimkelen`.
* *Ver los logs:* `docker logs -f kimkelen`.

## Conexión a la base de datos

Dado que el contenedor corre en un segmento de red diferente, la base de datos
deberá admitir conexiones desde la red. Es por ello que si existen problemas de
conexión a la base de datos, deberá verificar que la configuración sea la adecuada:

* Que los datos de nombre de base de datos, host, usuario y contraseña sean los
  esperados.
  * Los archivos de configuración pueden verse en el volumen montado por Docker
    bajo `data/config/`.
* Que se disponga de permisos para conectarse desde la red de Docker.
  * Si el problema es de permisos accediendo desde otro host, verificar que:
    * El servidor de MySQL esté escuchando en una IP válida (no en 127.0.0.1).
    * Que el usuario tenga permisos de acceso:

```
GRANT ALL PRIVILEGES ON kimkelen_docker.* to kimkelen@'%' identidied by 'muysecreta';`
```

## Los volúmenes

El contenedor de Kimkelen provee dos volúmenes:

* `code/`: donde se almacenan las diferentes versiones de Kimkelen. El directorio
  contendrá las distintas versiones que vayan descargándose. Ver *GIT_REVISION*. 
* `data/`: configuraciones y datos persistentes entre las diferentes
  versiones instaladas. Aquí se mantienen:
  * `data/php.ini`: configuración de PHP.
  * `data/config/databases.yml`: configuración de la base de datos.
  * `data/config/propel.ini`: configuración de la base de datos.
  * `data/config/app.yml`: configuración de la aplicación básica.
  * `data/data/`: datos donde se almacenan documentos subidos al sistema.
  * `data/log/`: logs de la aplicación Symfony (no del web server).
  * `data/web/uploads`: otros uploads de la aplicación.

Un volumen en docker, permite persistir los datos del contenedor. Para más
información ver [la sección de Volúmenes del
manual](https://docs.docker.com/userguide/dockervolumes/).

La aplicación podrá correrse con un usuario estándar si se especifica la
variable de ambiente *USER_ID*.

## Interacción con el contenedor

### Reinstalando Kimkelen

Si se desea reinstalar la versión de Kimklen y/o reconfigurarlo, es posible
correr el contenedor con el comando siguiente:

```
docker run \
    -e USER_ID=`id -u` \
    -e DB_NAME=kimkelen \
    -e DB_USER=kimkelen \
    -e DB_PASS=muysecreta \
    -v /tmp/kimkelen/codigo:/code \
    -v /tmp/kimkelen/data:/data \
    --rm \
    cespi/kimkelen --reinstall
``` 

Este comando lo que hace es:

* Reinstala la versión específica de Kimkelen (la indicada por GIT_REVISION)
* Vuelve a crear los archivos:
  * `config/propel.ini`
  * `config/databases.yml`
  * `config/nc_flavor.yml`

Notar que no utilizamos `--name` y utilizamos `--rm`. Esto lo que hace es crear
un nuevo contenedor que sólo actualizará los mismos volúmenes que utiliza el
contenedor Kimkelen. *Asumimos existe un contenedor que ya se encuentra
corriendo y comparte los volúmenes mencionados*


### Version con datos de demo

Para correr una instancia de kimkelen con datos de prueba:

```
docker run \
    -e USER_ID=`id -u` \
    -e DB_NAME=kimkelen \
    -e DB_USER=kimkelen \
    -e DB_PASS=muysecreta \
    -v /tmp/kimkelen/codigo:/code \
    -v /tmp/kimkelen/data:/data \
    --rm \
    cespi/kimkelen --demo
```

Notar que no utilizamos `--name` y utilizamos `--rm`. Esto lo que hace es crear
un nuevo contenedor que sólo actualizará los mismos volúmenes que utiliza el
contenedor Kimkelen. *Asumimos existe un contenedor que ya se encuentra
corriendo y comparte los volúmenes mencionados*

Los datos de acceso en la demo son: usuario *admin* contraseña *admin*

### Comandos symfony

Este contenedor permite correr comandos Symfony dentro del contenedor de la
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

*Por ejemplo, para cambiar el estilo visual del contenedor podría usarse:*

```
docker run \
    -e USER_ID=`id -u` \
    -v /tmp/kimkelen/codigo:/code \
    -v /tmp/kimkelen/data:/data \
    -it \
    --rm \
    cespi/kimkelen \
    symfony kimkelen:flavor bba
```

### Acceder a un shell en el contenedor

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

El contenedor mínimamente deberá definir el puerto en el que ejecutará el
servicio.

### Definir el puerto

Se exponen los puertos 80 y 443. La forma de asociar estos puertos con puertos
de la máquina local es con la opción `-p PTO_LOCAL:PTO_DESTINO`, por ejemplo `-d
8000:80` indica que al acceder al puerto 8000 de la PC donde se corre el
contenedor redireccionará al puerto 80 del contenedor, donde se encuentra
ejecutando Kimkelen.

### Definir los volúmnes

Pueden utilizarse volúmenes que maneja Docker o directorios de la PC donde se
ejecuta Docker. Si se utiliza la opción:

```
    -v `pwd`/code:/code -v `pwd`/data:/data
```

Estamos asociando los directorios `/code` y `/data` del contenedor con dos
directorios de la PC local ubicados en el directorio actual (por ello pwd) con
los mismos nombres respectivamente.

### Especificar el usuario USER_ID

Si corremos el contenedor sin especificar un usuario, todas las operaciones se
realizan con el usuario www-data que en Debian es el UID 33. Esto nos complicará
la interacción con las configuraciones alojadas en los volúmenes dado que necesitaremos
privilegios de root para modificar estos archivos.
Para simplificar esta cuestión, se provee la posibilidad de indicar con qué UID
deberá correr Kimkelen. Esto es posible con la opción `-e USER_ID=1000` o por
ejemplo

```
    -e USER_ID=`id -u`
```

Que seteará esta variable al UID del usuario actual.

### Datos de conexión a la base de datos

Pueden definirse los siguientes valores: DB_NAME, DB_HOST, DB_USER, DB_PASS

Las variables de entorno mencionadas se pasan al contenedor de la siguiente
forma:

```
    -e DB_NAME=kimkelen_prueba
```

*Es importante destacar que una vez creados los archivos `config/databases.yml`
y `config/propel.ini` estas variables serán ignoradas. Si se desea volver a
crear estos archivos deberá eliminarlos y ejecutar nuevamente el contenedor*.

### Definir la versión de Kimkelen: GIT_REVISION

El contenedor instalará la versión disponible en Master, pero puede cambiarse
por otra definiendo la variable *GIT_REVISION* de la siguiente forma:

```
    -e GIT_REVISION=v2.19.4
```

## Actualizando el contenedor

Para obtener la última versión de esta imagen ejecutar: 

```
docker pull cespi/kimkelen
```
