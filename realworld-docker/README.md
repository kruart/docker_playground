Before running the app you need to add local domain to `/etc/hosts`
```.shell script
sudo nano /etc/hosts

# add this line to /etc/hosts file for prod (see nginx.conf.prod server_name)
127.0.0.1       realworld-docker.com
# add this line to /etc/hosts file for dev (see nginx.conf.local server_name)
127.0.0.1       realworld-docker.local
```
#### some commands to work with project  
`docker build --file=frontend.dockerfile  -t realword-docker-frontend .`  # run container one by one (not recommended)  
`docker build --file=backend.dockerfile  -t playground-web-backend .`     # actually it's log from letbo  
`docker ps -a` # show containers    
`docker images -a` # show images    
`docker-compose build` # build all containers  
`docker rm daaf` # remove container  
`docker rmi $(docker images -q) --force` # remove all images    
`docker-compose up`  # run containers    
`docker-compose up --build` # run containers if need rebuild them  
`docker logs docker-playground_api_db_1` # loot at logs that specific container generated  
`docker volume`  help  
`docker volume ls` list of volumes  
`docker-compose -f docker-compose.yml -f docker-compose.development.yml up --build` # run dev configuration  
`docker exec -it docker-playground-api sh` # run shell in interactive mode      

#### cheet sheet commands
##### docker container
`docker container create -p 80:80 --name proxy nginx` create container  
`docker container start proxy` then we could start above container  
`docker container run nginx` create and run nginx container  
`docker container run -d nginx` create and run nginx container in background mode (-d or --detach)  
`docker container run -d -p 80:80 nginx` create and run nginx container in background mode(-d) and with mapped port to host machine(-p)  
`docker container run -d -p 80:80 --name proxy nginx` -d - in background mode; -p - mapped port to host machine; --name container name (proxy in this ex.)  
`docker container run -d -p 80:80 --name proxy --health-cmd 'curl http://localhost:80/' --health-retries 3 --health-interval '1s' nginx`  
`--health-cmd` - means which command we need to run to determine if container is health  
`--health-retries` - required number of tries to call health-cmd (default 3)    
`--health-interval` - interval 1 second (default 30s)  
To check if container is healthy or not use `docker container ls` and `docker container inspect <container_name>`  
`docker container run -d -p 80:80 nginx` create and run nginx container in interactive mode(-i) and with mapped port to host machine(-p)  
`docker container run -d -p 80:80 --name proxy -m 10485760 nginx` -m (alias for --memory) sets memory hard limit that container could use (limit in bytes. 10485760 = 10Mb (10 * 1024 * 1024)). Use `docker stats` to see memory usage  
`docker container run -d -p 80:80 --name proxy -m 10485760 --memory-reservation 5242880 nginx` --memory-reservation  allows you to specify a soft limit smaller than --memory which is activated when Docker detects contention or low memory on the host machine. If you use --memory-reservation, it must be set lower than --memory for it to take precedence. Because it is a soft limit, it does not guarantee that the container doesn’t exceed the limit.  
`docker container run -d -p 80:80 --name proxy -m 5242880 --memory-swap 10485760 nginx`  --memory-swap the amount of memory this container is allowed to swap to disk
`docker container run -d -p 80:80 --name proxy --restart always nginx` --restart always (restart policy)  
`no` — default  
`on-failure` — when error occurs  
`always` — always restart  
`unless-stopped` — until container won't be stopped  

`docker container run --rm -d -p 80:80 --name proxy nginx` remove container after container will be stopped  
`docker container run -i -t --name proxy nginx bash` -i -t add interactive terminal (Allocate a pseudo-TTY)  
`docker container run -it nginx bash` the same  
`docker [container] attach proxy` allow to attach to container that running in background (-d, --detach)  
```.shell
#!/bin/bash
# create and run docker container in background mode (--detach) and save container id to variable
CONTAINER_ID=$(docker container run -d -p 80:80 --name proxy nginx)

# docker container commit - create image based on some container 
docker container commit --author "My Name myname@gg.com" --message "Add curl" "$CONTAINER_ID" <acc-name>/nginx-curl:0.0.1
``` 
`docker container inspect proxy` get detailed info about container  
`docker container inspect proxy --format "IP: {{ .NetworkSettings.IPAddress }} | Gateway: {{.NetworkSettings.Networks.bridge.Gateway}}"` get formatted info about container (only what we need)  
`docker container rename proxy1 proxy` rename container  
`docker container rm proxy1 proxy2 proxy3 ` remove containers  
`docker container stats` get real-time resources info (how many cpu, memory container uses etc)  
`docker container prune` remove all stopped containers  
`docker container port proxy` get info about container ports   
`docker container logs db` shows container logs  
```
# copy files and dirs inside container
docker container cp ./data/*.html proxy:/usr/share/nginx/html
docker container cp ./data/css proxy:/usr/share/nginx/html/css

# shows differences/changes
`docker container diff proxy`
# A - means file or directory was added
# B - means file or directory was removed
# C - means file or directory was changed  
```

`docker container restart proxy` restart container  

##### docker images
Note: for some commands we have two syntax variations:  
- `docker image <command>` new syntax  
- `docker <command>`  old syntax

`docker images` show list of images, old syntax    
`docker image ls` show list of images, new syntax    
`docker pull nginx` download image on local machine  (`docker image pull` alternative command)  
`docker image tag nginx <acc-name>/nginx` create image copy and give a name for it  
`docker push <acc-name>/nginx` push image to user repository  
`docker image inspect nginx` show detailed image info    
`docker image inspect fholzer/nginx-brotli --format "{{.Config.Env}}"` show specific image info (formatted info)    
`docker image history nginx` show info about image layers    
```.shell
# example how to work with local registry
# 1. create and run local registry
# 2. run in background mode
# 3. 5000 local port is mapped on 5000 container port
# 4. restart always
# 5. set name my-registry
docker run -d -p 5000:5000 --restart always --name my-registry registry:2

# set new image tag/name (локальный registry)
docker image tag nginx localhost:5000/nginx

# push image to local registry
docker push localhost:5000/nginx

# The path where the images are stored in the local repository /var/lib/registry/docker/registry/v2
# to see pushed images use `docker exec -it my-registry sh` and go /var/lib/registry/docker/registry/v2 

docker pull localhost:5000/nginx # then we could pull the image from our local/remote registry  

# So the point is since docker allows only one private repository,  
# we always could create our own registry on some remote machine
# push there our custom images and reuse such registry like a private repository  
```

#### Docker volumes
Note: all volumes are stored in `/var/lib/docker/volumes` path by default  
`docker volume ls` volume list      
`docker volume rm <volume-name-or-hash>` rm volume        
`docker volume prune -f` remove unused volumes  
`docker volume inspece storage` inspect "storage" volume  
```.shell
# create volumes
docker volume create storage
docker volume create config

# mount storage and db to /data/db and /data/configdb paths inside container
docker container run -d -v storage:/data/db -v config:/data/configdb -p 27017:27017 mongo 
```
`docker container run -d -v storage:/usr/share/nginx/html:ro -p 80:80 nginx:alpine'`  :ro - read only  
```.shell
# create volume; allocate 100m for volume; device tmpfs
docker volume create --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=100m \
    storage

# container won't start, cause mongo size is ~ 400m, we allocate only 100m  
docker container run -d -v storage:/data/db --name mongo mongo
```
`docker container run -d --mount source=storage,target=/usr/share/nginx/html --name webhost nginx:alpine` alternative syntax how to create volumes
```
# type could be bind, volume, or tmpfs. Another syntax mount, type, source target
docker container run -d \
    --mount type=bind,source="$(pwd)"/html,target=/usr/share/nginx/html,readonly \
    --publish 80:80 \
    --name webhost nginx:alpine
```  

```
docker run -d \
  -it \
  --name webhost \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:alpine

#OR
docker run -d \
  -it \
  --name webhost \
  -v "$(pwd)"/target:/app:ro \
  nginx:alpine
```



#### Docker NETWORK  
Note: one container could belongs to several networks  
----
`docker network rm frontend` remove frontend network  
`docker network create frontend` create frontend network (bridge)  
`docker network create -d bridge frontend` the same  
`docker network ls` network list  
`docker network connect frontend proxy`  connect container `proxy` to `frontend` network  
`docker network disconnect frontend proxy` disconnect container from network    
`docker container inspect proxy" print network info  
`docker container inspect proxy --format "{{ .NetworkSettings.Networks }}" print network info  
`docker container run -d -p 80:80 --name proxy --net frontend nginx` run container and add network(all in one command)  
`docker network prune` remove unused networks  
```.shell
#!/bin/bash

# create network
docker network create frontend

# run two containers with network frontend and network alias is 'search'
docker container run -d --net frontend --net-alias search elasticsearch:2
docker container run -d --net frontend --net-alias search elasticsearch:2

# nslookup (balancer, robin round)
docker container run --rm --net frontend alpine nslookup search

# curl server info
# docker container run --rm --net frontend centos curl -s search:9200
```

#### Docker system
`docker system prune` # Remove all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes.  
`docker system prune -a -f` -a (all), -f (force)  
`docker system df` displays information regarding the amount of disk space used (images, containers, volumes, cache)  
  
