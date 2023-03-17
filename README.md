# aardvark

## Intro
Main location for a project that aims to create a product that helps connect people with common interests near me so I can make connections wherever I am.  

Wiki contains information for the product
This repo contains the files needed to construct the application from many repo's

## Pre-requisite  
This has only been tested on a dev machine that is setup in the guide [here](https://github.com/fistralpro/dev_setup)

## First Run

Clone this repo into your linux environment 

//TODO run script that reads from components file and git clones into directory, then runs each directories startup script for development and deployment

Bring up all the docker containers required to run the application
`docker compose up`  

**or** add them to a swarm
Deploy the web apps image (defined in the docker-compose file) to the registry  
`docker compose push`    

Now we can deploy the stack to the swarm  
`docker stack deploy --compose-file docker-compose.yml aardvark`

check it is running  
`docker stack services aardvark`

note - while working from the swarm, localhost from windows no longer works; you will need to connect to the eth0 network     
To find the eth0:   
`ip addr show eth0 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1`

## Clean up

Remove swarm  
`docker stack rm aardvark`  
or scale individual service to 0  
`docker service update --replicas 0 aardvark_postgis`
`docker service update --replicas 0 aardvark_tomcat`

Delete volumes  
`docker volume prune`

Delete logs  
`?`

## Architecture
### swarm
In windows you may need to connect by ip address (replace localhost) 
`ip addr show eth0 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1`  

host | Service | Purpose 
--- | --- | ---
localhost:5432 | PostGIS | Data storage
localhost:5672 | RabbitMQ | Messaging bus 
localhost:15672 | RabbitMQ | Messaging bus GUI 
localhost:8081 | Tomcat | Backend server
localhost:8080 | Nginx | Load balancer for both frontend servers
localhost:80 | Nginx | Frontend server 1
localhost:80 | Nginx | Frontend server 2
localhost:? | Keycloak | Identity and Access Management
localhost|? | ? | logging


## Checking infrastructure
### postgis
Connect to db using your favourite client
### rabbitmq notes
We can see whether rabbit is up and if any vhosts have been created with  
`curl -i -u guest:guest http://localhost:15672/api/vhosts`

### tomcat
If you want to run a tomcat war/jar you can copy the archive to the volume (symlinks don't work) the default location is the CATALINA_HOST

```
sudo cp "$(pwd)/sample/tomcat8/webapps/sample.war" /var/lib/docker/volumes/aardvark_tc_app/_data/webapps/sample.war"
```
you should see a sample page at localhost:8081/sample/


### nginx
Nginx has been setup as a load balancer and two servers... TODO - sort out other things

### docker volumes
`docker volume ls`  

`docker volume inspect aardvark_tc_app`  

## TODO
Backup DB
Env Variable
Logging
HTTPS

### notes
Ubuntu doesn't have a root, so instead of su - 
`sudo su`

Interactive shell into container
`docker exec -it [id] sh`
