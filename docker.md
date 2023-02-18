# Docker tips
- Guide to install Docker on Raspbian - https://docs.docker.com/engine/install/debian/
### Clean up space used by Docker
WARNING! This will remove:                                                                                                          
- all stopped containers
- all networks not used by at least one container
- all images without at least one container associated to them
- all build cache
````
docker system prune -a
````
### Show docker disk usage
````
docker system df 
...
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          0         0         0B        0B
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B
````
### Attach to shell of a running container
If your container is running a webserver, for example, docker attach will probably connect you to the stdout of the web server process. It won't necessarily give you a shell.
````
docker exec -it <mycontainer> sh
````
### Docker Compose to Run Multiple Instances of a Service
```
docker-compose up --scale app=2
```
docker-compose.yml example:
````
version: "3"

services:

  app:
    build: app/
    command: node app.js
    expose:
      - "3000"
  nginx"
    build: nginx/
    command: nginx -g 'daemon off;'
    ports:
      - "80:80"
    depends_on:
      - app
````
This will create network with the default driver (bridge). So, nginx will round robin to each instance inside network and can be accessible by port 80 on the host. Nginx round robin configuration can be found here https://github.com/Frodo-Web/frodo-tips/blob/main/nginx-tips/nginx.md
