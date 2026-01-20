---

# 🚀 Despliegue de Astro con Docker y Traefik

Esta guía detalla cómo configurar un proyecto de **Astro** utilizando **Docker** para contenedores y **Traefik** como proxy inverso.

## 1. Preparación del Entorno

### Gestión de Node.js

Para evitar conflictos de versiones, recomendamos usar [fnm (Fast Node Manager)](https://github.com/Schniz/fnm). Una vez instalado, prepara tu proyecto:

```bash
# Instalar e instalar la versión LTS de Node
fnm use --install-lts

# Crear el nuevo proyecto de Astro
npm create astro@latest

```

---

## 2. Configuración de Docker

Para que el proyecto funcione correctamente, necesitamos dos archivos en la raíz del proyecto: el `Dockerfile` (receta de la imagen) y el `docker-compose.yml` (orquestación).

### A. Dockerfile (Multi-stage Build)

Usamos un proceso de dos etapas para que la imagen final sea ligera y segura.

```dockerfile
# Etapa 1: Construcción (Build)
FROM node:lts-slim AS build
WORKDIR /app

# Instalar dependencias
COPY package*.json ./
RUN npm install

# Copiar archivos y generar el sitio estático
COPY . .
RUN npm run build

# Etapa 2: Servidor de producción (Nginx)
FROM nginx:alpine
LABEL maintainer="tu-nombre"

# Copiamos los archivos generados desde la etapa 'build'
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

```

### B. Docker Compose

Este archivo gestiona el contenedor y su comunicación con el proxy **Traefik**.

> **Nota:** Antes de levantar el servicio, asegúrate de tener creada la red externa:
> `docker network create web`

```yaml
services:
  astro-app:
    build: .
    container_name: astro_container # Nombre del contenedor
    networks:
      - web
    labels:
      - "traefik.enable=true"
      # Define aquí tu dominio o host local
      - "traefik.http.routers.astro-router.rule=Host(`astro.localhost`)"
      - "traefik.http.routers.astro-router.entrypoints=web"
      # Puerto interno de Nginx definido en el Dockerfile
      - "traefik.http.services.astro-service.loadbalancer.server.port=80"

networks:
  web:
    external: true

```

---

## 3. Comandos Útiles

Para poner en marcha tu aplicación, ejecuta los siguientes comandos en tu terminal:

| Acción | Comando |
| --- | --- |
| **Levantar el contenedor** | `docker-compose up -d --build` |
| **Detener el servicio** | `docker-compose down` |
| **Ver logs en tiempo real** | `docker logs -f astro_container` |

---
