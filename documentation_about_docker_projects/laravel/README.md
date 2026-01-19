Esta es una documentación para poder crear un proyecto de laravel y configurarlo para que funcione con traefik

( 1 ) primero que nada crea tu proyecto laravel con el siguente comando

>>>>>>>>>

docker run --rm \
    -u $(id -u):$(id -g) \
    -v $(pwd):/app \
    composer:latest \
    composer create-project laravel/laravel:^12.0 mi-proyecto-laravel

>>>>>>>>>

como puedes ver el comando utiliza una imagen de composer:lates para crear el proyecto sin necesiad de instalar la tecnologia como paquete dentro de linux.

tambien se muestra el comando para crear el proyecto y especificar la vercion "^12.0" y tambien el nombe del proyecto que es "mi-proyecto-laravel".


( 2 ) una ves creado el proyecto, tienes que ejecutar los siguentes 2 comanos dentro de la raiz del poyecto para poder darle permisos a tu linux para editar los archivos de cache y el sqlite, ya que los proyectos creados con contenedores de lararvel son creados por el usuario de docker y no tu usuario combencional.

>>>>>>>>>

sudo chmod -R 777 storage bootstrap/cache
sudo chmod -R 777 database/

>>>>>>>>>

( 3 )  tienes 2 opciones, o puedes copiar la carpeta y su contenido llamada Docker dentro de la rais del proyecto, o puedes crear los archivos manualmente.

recuerda revisarr el documento porque hay que hacer una configuración en los archivos docker-compose.yml y el Dockerfile

crea la carpeta Docker y ingresa en ella
>>>>>>>>>
 	mkdir Docker
	cd Docker
>>>>>>>>>


(3.1)Crear los siguentes archivos 
>>>>>>>>>
	touch docker-compose.yml Dockerfile nginx.conf
>>>>>>>>>


(3.2)primero seria editar el archivo docker.compose.yml 
>>>>>>>>>
    nvim docker-compose.yml
>>>>>>>>>

el contendio de este archivo es el siguente:




>>>>>>>>>

name: laravel-2

services:
  # El "motor" de PHP
  app:
    build:
      context: ..
      dockerfile: Docker/Dockerfile
    container_name: laravel-2-app
    volumes:
      - ..:/var/www
    networks:
      - web

  # El servidor Web (Nginx) que conecta con Traefik
  webserver:
    image: nginx:alpine
    container_name: laravel-2-web
    volumes:
      - ..:/var/www
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    networks:
      - web
    labels:
      - "traefik.enable=true"
      # Usamos la variable para el dominio
      - "traefik.http.routers.laravel-2.rule=Host(`laravel-2.localhost`)"
      - "traefik.http.routers.laravel-2.entrypoints=web"
      - "traefik.http.services.laravel-2.loadbalancer.server.port=80"
      - "traefik.docker.network=web"

networks:
  web:
    external: true

>>>>>>>>>

El archivo docker-compose.yml sirve para indicar los servicios que usaremos para el proyecto y la conexion que tendremos con traefik

es importante que mies en cada lugar donde dice larave-1 y lo modifiques por el nombre de tu proyecto comunmente, pero lo mas importante es que no se repita con otos proyectos.

tambien en el label "traefik.http.routers.laravel-2.rule=Host(`laravel-2.localhost`)" puedes colocar el dominio que quieras, siempre y cuando el dominio o sub dominio apunten a tu servidor donde se encunetre el traefik. 

mas adelante hablaremos de el nombre del contenedor laravel-2-app asi que tenlo en cuenta.


(3.3) Edita el archivo Dockerfile
>>>>>>>>>
    nvim Dockerfile
>>>>>>>>>

este archivo lo usamos para configurar la imagen de php ya que por defecto no viene con todo lo que necesitamos para laravel, por eso le hacemos modificaciónes mediante este archivo.

el contenio es el siguente:

>>>>>>>>>
FROM php:8.4-fpm

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    git curl libpng-dev libonig-dev libxml2-dev zip unzip

# Instalar extensiones de PHP necesarias para Laravel
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Directorio de trabajo
WORKDIR /var/www

# Copiar el proyecto
COPY . /var/www

# Dar permisos a la carpeta storage (vital en Laravel)
# Cambia la línea 17 por esta:
RUN chown -R www-data:www-data /var/www/storage /var/www/bootstrap/cache

EXPOSE 9000
CMD ["php-fpm"]
>>>>>>>>>



(3.4) editar el archivo nginx.conf

>>>>>>>>>
    nvim nginx.conf
>>>>>>>>>

Dentro de este archivo pegaremos lo siguente
>>>>>>>>>
server {
    listen 80;
    index index.php index.html;
    root /var/www/public; # Laravel sirve desde /public

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass laravel-2-app:9000; # Nombre del servicio en docker-compose
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
>>>>>>>>>

tomr en cuenta que tienes que modificar done ice larravel-2-app por el nombre del contenedo que colocaste en la linea mencionada en el archivo docker-compose.yml la cual es  container_name: laravel-2-app en el servicio de app osea el primer serrvicio, el nombre que coloques como nombre de contenedorr es el mismo que necesita nginx, esto con el fin de evitar errores como lo podria ser que las peticiones de una app se envien a otra.

( 4) por ultimo debes de ejecutar el siguente comando para que se creen las imagenes y contenedores.

>>>>>>>>>
 docker compose up -d
>>>>>>>>>


