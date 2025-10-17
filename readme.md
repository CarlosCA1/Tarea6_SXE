## Explicación Docker Compose


````
services:
 db:
   image: mysql:8.0
   restart: always
   environment:
     MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
     MYSQL_DATABASE: ${MYSQL_DATABASE}
     MYSQL_USER: ${MYSQL_USER}
     MYSQL_PASSWORD: ${MYSQL_PASSWORD}
   volumes:
     - db_data:/var/lib/mysql
   healthcheck:
     test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "${MYSQL_USER}", "-p${MYSQL_PASSWORD}"]
     interval: 10s
     timeout: 5s
     retries: 5
     start_period: 10s


 prestashop:
   image: prestashop/prestashop:8.0
   depends_on:
     db:
       condition: service_healthy
   restart: always
   ports:
     - "8085:80"
   environment:
     DB_SERVER: ${DB_SERVER}
     DB_NAME: ${DB_NAME}
     DB_USER: ${DB_USER}
     DB_PASSWORD: ${DB_PASSWORD}
   volumes:
     - prestashop_data:/var/www/html


 phpmyadmin:
   image: phpmyadmin/phpmyadmin
   depends_on:
     db:
       condition: service_healthy
   restart: always
   ports:
     - "8086:80"
   environment:
     PMA_HOST: ${PMA_HOST}
     PMA_USER: ${PMA_USER}
     PMA_PASSWORD: ${PMA_PASSWORD}


volumes:
 db_data:
 prestashop_data:
````


Este archivo cuenta con tres servicios: db (una base de datos MySQL), Prestashop (la aplicación web) y phpMyAdmin (una interfaz gráfica para gestionar la base de datos).


Cada servicio es un contenedor y "volumes" (al final) declara oficialmente en el Docker Compose los volúmenes en los que se guardan los datos para que no se pierdan. Permite la persistencia de información incluso si los contenedores se eliminan o reinician. Las variables se definen en el .env.




### Base de datos:


* Definimos la imagen que vamos a utilizar (en mi caso la versión 8.0 de MySQL).


* "Restarts always" permite que el servicio se reinicie automáticamente si se detiene o falla.


* En "environment" definimos el usuario creado para la aplicación con permisos sobre la base de datos, la contraseña y el nombre de la base de datos, así como la contraseña del usuario root (el usuario administrador principal de la base de datos, con todos los privilegios posibles).


* En el apartado "volumes" se define dónde se guardan los archivos de datos del MySQL.


* El healthcheck le dice a Docker cómo comprobar si un contenedor está realmente funcionando correctamente. Condition indica que los servicios (Prestashop o phpMyAdmin) solo deben iniciar cuando el contenedor db esté “saludable”:

Se ejecuta periódicamente un comando dentro del contenedor para verificar si el servicio está funcionando correctamente. Envía una solicitud al servicio para comprobar que puede responder y aceptar conexiones con un usuario y contraseña válidos.

"Interval" indica cada cuánto tiempo Docker ejecutará el healthcheck.

"Timeout" es el tiempo máximo que Docker esperará la respuesta del comando. Si el comando tarda más de 5 segundos, se considera que falló.

"Retries" indica cuántas veces Docker intentará ejecutar el healthcheck antes de marcar el contenedor como “no saludable”. En este caso, si falla 5 veces seguidas el contenedor pasa a estado unhealthy.

"Start_period" es el tiempo inicial que Docker da antes de empezar a hacer chequeos. Se usa porque algunos servicios (como MySQL) tardan un poco en arrancar. Durante esos primeros 10 segundos, si MySQL aún no responde, no se considera error.


### Prestashop:


* Definimos la imagen que vamos a utilizar (en mi caso la versión 8.0 de Prestashop).


* Determinamos la base de datos de la que depende.


* "Restarts always" permite que el servicio se reinicie automáticamente si se detiene o falla.


* Mapeamos el puerto 80 (en el que escucha por defecto) del contenedor al 8085 del host, por ejemplo.


* Definimos el nombre de nuestro servidor y el de la base de datos anterior, así como su usuario y contraseña.


* En el apartado "volumes" se define dónde se guardan los archivos de la aplicación



### phpMyAdmin:


* Definimos la imagen que vamos a utilizar.


* Determinamos la base de datos de la que depende.


* "Restarts always" permite que el servicio se reinicie automáticamente si se detiene o falla.


* Mapeamos el puerto 80 (en el que escucha por defecto) del contenedor al 8086 del host, por ejemplo.


* Definimos el nombre del servidor de nuevo y un usuario y contraseña para phpMyAdmin (usaré el superusuario de la base de datos).