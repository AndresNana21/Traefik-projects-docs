# Documentación sobre como configurar un proyecto de laravel con modules filament desde 0


## tomar en cuneta que 
* se tiene una carpeta llamada Docker en el mismo nivel de este README.md para que utilices estos archivos y no tengas que crearlos desde 0.

* en la documentaión se encuneta que palabras claves tienes que cambiar para que tus contenedores permitan desplegar tu proyecto.

* puedes encontrar comandos que puedes reutilizar en ocaciones especificas cuando estes programando en tu proyecto.

---

## pasos realizados

## (1) crear un proyecto de laravel

para crear el proyecto utilizamos una imagen de composer de la cual sacaremos un contenedor temporal que nos permitira especifcar que queremos crea.

Tomar en cuneta que puedes modificar la verción del proyecto en **^12.0** o puedes modificar el nombre del proyecto en **mi-proyecto-laravel.**

```Comand

docker run --rm \
    -u $(id -u):$(id -g) \
    -v $(pwd):/app \
    composer:latest \
    composer create-project laravel/laravel:^12.0 mi-proyecto-laravel

```
### (1.1 ingresar al proyecto

en los siguentes pasos necesitamos estar en la raiz del proyecto por lo que necesitas usar este comando para ingresar en el.

toma en cuneta que si cambiaste el nombre del proyecto tambien necesitas modificar el comando.

```Comand
cd mi-proyecto-laravel 
```
---
## ( 2 )  crear los archivos para Dockerfile , docker-compose.yml y nginx.conf

en la raiz de el proyecto de documentación encontraras una carpeta para que la copies y la reutilices en este proyecto nuevo de laravel.

### (2.1) copiar la carpeta Docker desde tu hubicacíon actual.

recuerda cambia la direccion de el proyecto de documentación.

```Comand
cp -r ~/Traefik-projects-docs/Docker .
```
### (2.2) crear los archivos de manera manual


Crear la carpeta Docker

```Comand
  mkdir Docker
```

Crear los archivos

* Dockerfile
```Comand
cat <<'EOF' > ./Docker/Dockerfile
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
    libzip-dev && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# --- Instalar Composer ---
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
COPY . /var/www

# Dar permisos a la carpeta storage y bootstrap/cache
RUN mkdir -p /var/www/storage /var/www/bootstrap/cache \
    && chown -R www-data:www-data /var/www/storage /var/www/bootstrap/cache

EXPOSE 9000
CMD ["php-fpm"]
EOF
```
---



* docker-coompose.yml

```Comand
cat <<'EOF' > ./Docker/docker-compose.yml
name: laravel-1

services:
  # El "motor" de PHP
  app:
    build:
      context: ..
      dockerfile: Docker/Dockerfile
    container_name: laravel-1-app
    volumes:
      - ..:/var/www
    networks:
      - web

  # El servidor Web (Nginx) que conecta con Traefik
  webserver:
    image: nginx:alpine
    container_name: laravel-1-web
    volumes:
      - ..:/var/www
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    networks:
      - web
    labels:
      - "traefik.enable=true"
      # Usamos la variable para el dominio
      - "traefik.http.routers.laravel-1.rule=Host(`laravel-1.localhost`)"
      - "traefik.http.routers.laravel-1.entrypoints=web"
      - "traefik.http.services.laravel-1.loadbalancer.server.port=80"
      - "traefik.docker.network=web"

networks:
  web:
    external: true
EOF
```


---

* nginx.conf
```Comand
cat <<'EOF' > ./Docker/nginx.conf
server {
    listen 80;
    index index.php index.html;
    root /var/www/public; # Laravel sirve desde /public

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass laravel-1-app:9000; # Nombre del servicio en docker-compose
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
EOF
```

---

## (3) configuración de los archivos Dockerfile, docker-compose.yml y nginx.conf

estos archivos necesitan ser configurados para evitar conflictos como nombres de contenedores repetidos o contenedoes no existens, tambien lables mal echos.

estas configuraciónes son en casos especificos por lo que si es de prueba no deberia de haber problemas.

### (3.1) configuración de el docker-compose.yml

este archivo contiene lo que serian los contenedores para php y nginx, se repite la palabra **laravel-1** en cada parte para que puedas modificar facilmente.


lo mismo es para los lables y el dominio **laravel-1.localhost** que necesitas modificar, lo mas recomendado es tener el nombre del software para que no se repita con otros software que tienen diferente nombre.

### (3.2) configuración de el nginx.conf


en caso de cambiar la palabra **laravel-3-app** que seria el nombre del contenedo de php tienes que cambiarlo tambien en el nginx.conf para que las peticiones lleguen correctamente desde el traefik.



## (4) desplegar los servicios de docker 


* ingresar a la carpeta de Docker
```Comand
cd Docker
```


* desplegar los servicios en segundo plano con docker compose  
```Comand
docker compose up -d
```

* salir de la carpeta Docker
```Comand
cd ..
```









## (5) ingresar en el contenedor de php con composer incluido para instalar librerias o usar los servicios de filament


si quieres utilizar alguna libreria o instalarla, vas a necesitar de **composer** un manejador de paquetes para php, esta tecnologia se encunetra en el contenedor de php, por lo que tendras que ingresar en ese contenedor con el fin de acceder a esta herramienta.


para ingresar al contenedor ejecuta este comando el cual te permitira ingresar en modo bash para ejecutar comanos como si fuera una terminar normal.


```Comand
docker exec -it laravel-1-app bash
```
### (5.1) utilizar la libreria de filament-modules de el creador savannabits

esta es una libreria para trabajar con otras 2 librerias que normalmente no son compatibles, las cuales son :

* Filament
* nwidart/laravel-modules

filament es una libreria para crear gestores de información **CRUD** de manera muy simples y pofreciónales.

laravel modules es una libreria que nos permite separar todo lo que necesita laravel para trabajar, como lo serian migracione, models, controllers , rutas y incluyendo la estructura de filament para que funcione.

el comano a utilizar para instalar esta librerria es el siguente.

```Comand
composer require coolsam/modules
```
### (5.2) configuración de el composer.json 

un punto adicional es que deves de agregar la siguente configuración en el archivo composer.json para permitirle a los composer.json de cada modulo ser considerados.

recuerda salir del contenedor de php ya es el proyecto real que tienes que modificar, docker se encarga de copiar el proyecto real dentro de los contenedores gracias a sus volumenes.

* nota: los compose.json permiten la configuración o interación entre los archivos bases de laravel.


* lo que tienes que integrar
```Context
 "merge-plugin": {
        "include": [
            "Modules/*/composer.json"
        ]
    }
```


* Como tiene que quedar
```Result
 "extra": {
        "laravel": {
            "dont-discover": []
        },
        "merge-plugin": {
            "include": [
                "Modules/*/composer.json"
            ]
        }
    },

```
### (5.3) ejecutar la libreria

recuerda estar dentro del contenedor para que estos comandos funcionen correctamente.

```Comand
php artisan modules:install
```

### (5.4) ejecutar estilos de filament

a la hora de utilizar filament puede que no te carguen los estilos por lo que estos comandos te solucionan ese problema al publicar los estilos que filament necesita.

* dentro del contenedor
```Comand
php artisan filament:assets
php artisan storage:link
```












