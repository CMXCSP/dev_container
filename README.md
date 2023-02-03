# Instrucciones para montar el servidor en docker

Crear o modificar el archivo __.env__ con el siguiente contenido
```
# path para la carpeta de proyectos ejem: /home/usuario/workspace/ 
WORKSPACE=/c/laragon/www 
# nombre de usuario que ocupa el contenedor, 
# dentro del contenedor es la carpeta home del usuario donde estara el proyecto
DEFAULT_USER=username 
TIME_ZONE=America/Mexico_City
```

Dentro de la raiz de esta carpeta ejecutar.
``` docker compose up -d ``` __o__
``` docker compose -f docker-compose-with-mongo.yml up -d ```
Si aparece un error borrar la carpeta **.docker** en la carpeta raiz del usuario

Entramos al contenedor del motor de la base de datos _mariadb_, para crear interactivamente con la terminal la base de datos que requerimos
``` docker exec -it dev_container-mariadb-1 mariadb -h localhost -u root -p mysql ```

Entramos al contenedor _development_ para utilizar su terminal
``` docker exec -it development bash ```
y crear los enlaces simbólico al workspace, instalar composer y las dependencias para el funcionamiento del sitio si fuera necesario o bien comprobar que existen.


En la terminal nos encontramos en el **$HOME** del usuario, observamos la carpeta workspace con ``` ls -la ``` la cual debe mostrar el contenido de la ruta **WORKSPACE** definida en el __.env__. Y en la ruta **/var/www/html** debemos tener el enlace al workspace desde la carpeta del servidor web. Por lo que podemos ejecutar:
```
ls -la workspace/
ls -la /var/www/html
ln -s /home/username/workspace/REPOSITORIO_DE_TRABAJO/public/ /var/www/html/REPOSITORIO_DE_TRABAJO
```

Considere que REPOSITORIO_DE_TRABAJO es el nombre del directorio raiz donde esta almacenado el proyecto web, y corresponde al mismo directorio del anfitrión. Por lo que se debe crear el archivo __.env__ en la raiz, con las variables de entorno. **DB_HOST** corresponde a la **IPAddress** del contenedor de mariadb mostrada por ``` docker inspect dev_container-mariadb-1 ``` puede usar ``` docker inspect dev_container-mariadb-1 | grep "IPAddress" ``` para mostrar la IP a usar.
```
DB_HOST="172.18.0.2"
DB_NAME="dbname"
DB_USERNAME="dbusername"
DB_PASSWORD=""
```

Dentro de la misma raiz encontramos el archivo __composer.json__ para proceder con la instalacion de dependencias. Considera que debe estar dentro del workspace del huesped ``` cd /home/username/workspace/REPOSITORIO_DE_TRABAJO/ ```.
```
composer install
composer dump-autoload
composer dbinstall
```


Ahora revisamos la configuracion de apache, para configurar los host virtuales. Debe usar privilegios de superusuario ```su```, para copiar la salida de _STDO_ a otro archivo __*.conf__ y poder editarlo. 
```
ls -la /etc/apache2/
cd /etc/apache2/sites-available/
cat 000-default.conf > REPOSITORIO_DE_TRABAJO.conf
```

Se deben editar las siguientes lineas, en el archivo [REPOSITORIO_DE_TRABAJO.conf](REPOSITORIO_DE_TRABAJO.conf) se puede observar un ejemplo:
 - #ServerName www.example.com
 - ServerAdmin webmaster@localhost
 - DocumentRoot /var/www/html

Tambien revisamos con ```cat /etc/hosts``` que se encuentre la **IPAddress** del servidor _development_ en dicho archivo. Y salimos de la terminal del contenedor.

Reiniciamos los contenedores con ``` docker compose restart ```

Y podemos probar el proyecto en [localhost/REPOSITORIO_DE_TRABAJO](http://localhost/REPOSITORIO_DE_TRABAJO/) 


## Consulta de estado de contenedores
Podemos consultar que contenedores e imagenes existen con:
``` docker ps -a ``` __o__ 
``` docker compose ps -a ```

Consultamos la configuracion de variables del contenedor, particularmente nos interesa **IPAddress**
``` docker inspect development-mariadb-1 ```
``` docker inspect development ```

Levantar ``` docker compose up -d ```
Encender ``` docker compose start ```
Apagar ``` docker compose stop ```


## Notas
 - No olvide Cambiar _REPOSITORIO_DE_TRABAJO_ por el nombre la carpeta donde se almacena el proyecto.
 - Agregar o cambiar la direccion IP de la base de datos en el archivo __.env__ del proyecto
 - El password del usuario root para el contenedor es _'asdf1234'_ puede cambiarse en el [Dockerfile](Dockerfile)
 - El password del usuario root para el contenedor de mariadb es _'root'_ puede cambiarse en el [docker-compose.yml](docker-compose.yml) o [docker-compose-without-mongo.yml](docker-compose-without-mongo.yml)

