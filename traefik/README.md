# Doc para configurar tu traefik 

por temas de compatibilidad con distintos despligues locales se usa el puerto 8082 para resivir las peticiones y no el 80 que seria el que normalmente se usa, en caso de despligue cambia todos los lugares donde dice 8082 por 80.

el nombre "web" es la red que usaran todos los contenedores para conectarse a traefik y poder tener un trafico http/s entre los navegadores webs y los contenedores.

en caso de que quieras tener diferentes traefik recuerda especificar puertos distintos para dashboard y puertos para apps, junto con un container_name diferente para cada traefik y el name: architecture tambien tiene que ser cambiado.

si quieres un traefik con dashboard puedes utilizar este pero no es recomendado en producion.

```traefik
name: architecture
services:
  traefik:
    image: traefik:v3.6
    container_name: traefik
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"

      # EntryPoint para apps
      - "--entrypoints.web.address=:8082"

      # Dashboard clásico en 8080
      - "--api.dashboard=true"
      - "--api.insecure=true"
    ports:
      - "8081:8080"   # Dashboard
      - "8082:80"   # Apps
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - web

networks:
  web:
    name: web
  
```
si prefieres un traefik mas seguro puedes utilizar este que no incluye acceso a el dashboard de traefik mas recomendado para prouccion.



```traefik
name: architecture
services:
  traefik:
    image: traefik:v3.6
    container_name: traefik
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"

      # EntryPoint para apps
      - "--entrypoints.web.address=:8082"
    ports:
      - "8082:80"   # Apps
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - web

networks:
  web:
    name: web
  
```

