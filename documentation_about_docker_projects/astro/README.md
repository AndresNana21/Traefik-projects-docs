para poder crear el docker-compose.yml y el Dockerfile para proyectos de astro sigue estos pasos.

( 1 )  te recomendamos instar fnm para gestionar las verciones de node, una ves lo tengas crea un proyecto de astro con el siguente comando.

npm create astro@latest



el docker-compose.yml tiene que verse asi solo cambia los lugares donde se usa la palabrar "astro 2"
>>>>
  
services:
  astro-app:
    build: .
    container_name: astro2
    networks:
      - web
    labels:
      - "traefik.enable=true"
      # Configura aquí el host que quieras usar, o usa uno local para pruebas
      - "traefik.http.routers.astro2.rule=Host(`astro2.localhost`)"
      - "traefik.http.routers.astro2.entrypoints=web"
      # Traefik enviará el tráfico al puerto 80 del contenedor de Nginx
      - "traefik.http.services.astro2.loadbalancer.server.port=80"

networks:
  web:
    external: true # Si ya creaste la red 'web' previamente

>>>>

y el Dockerfile se tiene que ver asi 

>>>>>
# Etapa 1: Construcción
FROM node:lts-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Etapa 2: Servidor estático
FROM nginx:alpine
# Copiamos el build de Astro a la carpeta que sirve Nginx
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
>>>>>
