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