Before running the app you need to add local domain to `/etc/hosts`
```.shell script
sudo nano /etc/hosts

# add this line to /etc/hosts file for prod (see nginx.conf.prod server_name)
127.0.0.1       realworld-docker.com
# add this line to /etc/hosts file for dev (see nginx.conf.local server_name)
127.0.0.1       realworld-docker.local
``` 


####Commands
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
`docker system prune` # Remove all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes.   
