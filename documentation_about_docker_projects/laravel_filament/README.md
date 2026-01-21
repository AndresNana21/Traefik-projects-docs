# Documentación para poder desplegar un proyecto de laravel con filament

Tienes que tener una cacrpeta llamada Docker en la cual vas a tener los isguentes archivos DockerFile Docker-compose.yml 
nginx.conf
```Comands

mkdir Docker

touch Docker/docker-compose.yml
touch Docker/Dockerfile
touch Docker/nginx.conf

```
---


## Configurar el docker-compose.yml

este es un archivo el cual permite especificar que servicios, contenedores lables de traefik se van a utilizar.

lo que tienes que tomar en cuneta es modificar todo lugar donde se encunetre la palabra "laravel-3" por el nombre de tu proyecto para no tener problemas de compatibilidad

```docker-compose.yml
name: laravel-3

services:
  # El "motor" de PHP
  app:
    build:
      context: ..
      dockerfile: Docker/Dockerfile
    container_name: laravel-3-app
    volumes:
      - ..:/var/www
    networks:
      - web

  # El servidor Web (Nginx) que conecta con Traefik
  webserver:
    image: nginx:alpine
    container_name: laravel-3-web
    volumes:
      - ..:/var/www
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    networks:
      - web
    labels:
      - "traefik.enable=true"
      # Usamos la variable para el dominio
      - "traefik.http.routers.laravel-3.rule=Host(`laravel-3.localhost`)"
      - "traefik.http.routers.laravel-3.entrypoints=web"
      - "traefik.http.services.laravel-3.loadbalancer.server.port=80"
      - "traefik.docker.network=web"

networks:
  web:
    external: true
```
--

## Configurar el archivo Dockerfile


Es un archivo usado dentro de docker-compose.yml , en este archivo especificamos como es que se tiene que crear la imagen de php, para que esta imagen vengan con composer y con las extenciones necesarias para que filament funcione corrrectamente.

```Dockerfile
FROM php:8.4-fpm

# Instalar dependencias del sistema y librerías necesarias para extensiones de PHP
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    libicu-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libzip-dev

# ---  Instalar Composer ---
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Configurar e instalar extensiones de PHP necesarias para Laravel y Filament 4
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install \
    pdo_mysql \
    mbstring \
    exif \
    pcntl \
    bcmath \
    gd \
    intl \
    opcache \
    zip

# Directorio de trabajo
WORKDIR /var/www

# Copiar el proyecto
COPY .. /var/www

# Dar permisos a la carpeta storage y bootstrap/cache
RUN chown -R www-data:www-data /var/www/storage /var/www/bootstrap/cache

EXPOSE 9000
CMD ["php-fpm"]
```

## configurar el nginx.conf

es un archivo que nos permite configurar el contenedor de nginx parar que este contenedor pueda resivir peticiones http https correctamente y pueda devolver la informacion ya compliada gracias al contenedor de php que termina como app


lo que tienes que modificar en este archivo es donde se encunetra la palabra "laravel-3" por la palabra que usaste en docker-compose.yml parar que de esta manera nginx y php esten conectados sin ningun problema.

```nginx.conf
server {
    listen 80;
    index index.php index.html;
    root /var/www/public; # Laravel sirve desde /public

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass laravel-3-app:9000; # Nombre del servicio en docker-compose
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```





## configuración dentro del contenedor de app (php)

Tenemos que jecutar los siguentes comandos con el fin de que les permitas a tu docker interactuar con los archivos necesarios.


ingresarr al contenedor de php como el usaurio www-data
```comands
docker exec -it -u www-data [name-containter] bash
```
Dar permisos dentro del contenedor al usuario www-data

```comands
chown -R www-data:www-data /var/www/storage /var/www/bootstrap/cache

chmod -R 775 /var/www/storage /var/www/bootstrap/cache

```

##  permisos por comandos dentro de el contenedor.

los comandos que ejecutas como php "artisan make:filament-resource" dentro de el contenedor de php puede darte problemas de permisos ya que el que crea ese resource es el usuario de docker y no tu usuario normal de linux.

Comandos que se tienen que jecutar en la raiz de el proyecto en local cada ves que crees algo dentro de un contenedor, para poder editar estos archivos desde un usuairo de linux normal.

```Comands
sudo chmod -R 775 storage bootstrap/cache
sudo chown -R $USER:www-data .
```










