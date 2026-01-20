# Documentación para poder desplegar un proyecto de laravel con filament


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





# configuración dentro del contenedor de app (php)

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
