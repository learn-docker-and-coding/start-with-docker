# Install Docker
###### Remove possible old Docker versions and install the new or stable version
```
sudo apt-get remove docker docker-engine docker.io
```
```
sudo apt-get update
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
```
sudo apt-get update
```
```
sudo apt-get install docker-ce
```
```
sudo docker version
```
###### To execute Docker without `sudo`
```
sudo usermod -aG docker $(whoami)
```
# Docker commands
To see the Docker version:
```
docker version
```
To run one image (one list of statement to mount one container) just execute:
```
docker run "image-name"
```
With this, if the image not exist in the local image repository, the Docker pull from the docker hub: `http://hub.docker.com`

To list all container:
```
docker ps -a
```
To list all built local image:
```
docker images
```
To access the container:
```
docker run -it "docker_listed_image"
```
To start one container:
```
docker start "container_id"
```
To stop one container:
```
docker stop "container_id"
```
To start one container and attach it:
```
docker start -a -i "container_id"
```
To remove one container:
```
docker rm "container_id"
```
To remove all inactive containers:
```
docker container prune
```
To remove an image:
```
docker rmi "image"
```
To run container terminal  detached:
```
docker run -d "container"
```
To run container terminal  detached and request for one port(randomic) to access the application from local machine and after see the ports:
```
docker run -d -P "container"

docker port "container"
```
To run in specific port(not randomic) to access the application from local machine and after see the ports:
```
docker run -d -p [external_port]:[internal_port] --name [name_i_want_to_call_the_container] "container"

docker port [name_i_want_to_call_the_container]
```
To pass some environment variable (-e to modify the variable value in the container):
```
docker run -d -P -e [VARIABLE_NAME]="value"
```
To stop all containers without wait some time (-t 0). To see just the containers ids `docker ps -q`:
```
docker stop -t 0 $(docker ps -q)
```
# Using volumes
Volumes is a directory that reference to Docker host and not in the Docker container, so, if the container stop or be deleted, the data will not lost. If configure the reference to point from local directory to Docker volume, all data will be saved in that pointed directory in the local machine.
To run with volume:
```
docker run -v "[directory, example, /var/www]" "container"
```
To run with volume pointed to local directory:
```
docker run -v "[local directory]:[local container]" "container"
```
# Executing code 
* Node image
* Volume local (with or no [`$(pwd)`]) and volume container
* Mapping ports
* Point the initial container directory (`-w`) 
* Code in local machine and running inside the docker container

```
docker run -d -p 8080:3000 -v "[local path]:/var/www" -w "/var/www" node npm start

docker run -d -p 8080:3000 -v "$(pwd):/var/www" -w "/var/www" node npm start
```
# Creating Dockerfile and build an image
Dockerfile is a file that contain statements to execute when the image is mounting. Each statement mount layer containers, so, an image has one or more layers containers. This procedure allow reuse some layers containers by Docker manager when build more images.

Dockerfile can be named of different ways principally when exists in the same path more than one Dockerfiles. Always need the extension "dockerfile",  e.g `node.dockerfile`,  however if exists just one dockerfile in the path, keep the name Dockerfile.

Dockerfile example bellow do:
* Mount image based on latest Node version;
* Configure some port to expose the app;
* Copy all files in the path and move on to some container path;
* Define the work directory;
* Install the all libraries need;
* Start the server and expose the port to access the app outside the container;

``FROM node:latest
MAINTAINER [the_name_of_responsible]
ENV PORT=3000
ENV DIR=/var/www
COPY . $DIR
WORKDIR $DIR
RUN npm install
ENTRYPOINT npm start
CMD ["npm", "start"]
EXPOSE $PORT
``
Executing the command bellow, an image will be mounted and the name will be the value after `-t`.
```
docker build -f Dockerfile -t [username/image_name]
``` 

# Up the image to Docker hub
To upload some image to Docker hub is necessary create an account there. After that is need follow the steps:
1. Execute `docker login` and type the resquests;
2. Execute `docker push "image_name"`.

To pull some image, just need execute:

*  `docker pull "the_image_choised"`

# Docker networking
To create personal network to use:
```
docker network create --driver bridge "personal_network_name"
```
To run one container using that personal network:
```
docker run -it --name "name_of_container" --network "personal_network_name"
```
To see the configuration of one container:
```
 docker inspect "name_of_container"
```
# Docker composer
Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.
Docker compose is not installed by standard on OS Linux. To install it, execute:
```
sudo curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
```
sudo chmod +x /usr/local/bin/docker-compose
```

#### YAML file example
That YAML example was used by `Douglas Quintanilha`, teacher at Alura in the Docker course.

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"
        networks: 
            - production-network
        depends_on: 
            - "node1"
            - "node2"
            - "node3"

    mongodb:
        image: mongo
        networks: 
            - production-network

    node1:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-1
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

    node2:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-2
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

    node3:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-3
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

networks: 
    production-network:
        driver: bridge
```

#### Executing the YAML file
To build:
```
docker-compose build
```
To up the built containers:
```
docker-compose up -d
```
To list the containers running:
```
docker-compose ps
```
To execute some command in some container:
```
docker exec -it "container_1" ping "container_2"
```
To stop and remove together all containers:
```
docker-compose down
```

#### Balancer containers ilustrations
Image access in: https://psi.cx/2017/crafted-docker-reverse-proxy/

![containers](https://psi.cx/2017/crafted-docker-reverse-proxy/request-flow-diagram.png)


#### To more help

```
https://dzone.com/articles/top-docker-commands-itsyndicate
```
