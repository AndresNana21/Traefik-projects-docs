FROM php:8.4-fpm

# 1. Instalar dependencias del sistema
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

# 2. Instalar Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 3. Instalar extensiones de PHP
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

# 4. Directorio de trabajo
WORKDIR /var/www

# 5. Copiar el proyecto (OJO: Asegúrate de tener un .dockerignore para no copiar vendor)
COPY . /var/www

# 6. CONFIGURACIÓN CRÍTICA DE PERMISOS
# Creamos las carpetas necesarias, incluyendo la de la DB para SQLite
RUN mkdir -p /var/www/storage/framework/cache/data \
    && mkdir -p /var/www/storage/framework/app/cache \
    && mkdir -p /var/www/storage/framework/sessions \
    && mkdir -p /var/www/storage/framework/views \
    && mkdir -p /var/www/storage/logs \
    && mkdir -p /var/www/bootstrap/cache \
    && mkdir -p /var/www/database

# Crear el archivo sqlite si no existe para que el dueño sea www-data desde el inicio
RUN touch /var/www/database/database.sqlite

# Asignar dueño y permisos de forma recursiva
RUN chown -R www-data:www-data /var/www \
    && chmod -R 775 /var/www/storage /var/www/bootstrap/cache /var/www/database

# 7. Cambiar al usuario www-data para que los comandos ejecutados (como composer) 
# y el proceso PHP-FPM corran con sus permisos
USER www-data

EXPOSE 9000
CMD ["php-fpm"]
