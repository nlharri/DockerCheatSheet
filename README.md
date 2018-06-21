# Cheat Sheet for Docker
This is a cheat sheet for using Docker. Based on https://docs.docker.com/get-started/

## Test Docker version

```
docker --version
```

## View more details

```
docker info
```

or: 

```
docker version
```

## Avoid using ```sudo``` when running Docker commands

### Create ```docker``` group

```
sudo groupadd docker
```

### Add user to ```docker``` group

```
sudo usermod -aG docker $user
```

(After this it is necessary to logout and login again.)

### Test if docker commands can be run without ```sudo```

```
docker run hello-world
```

### Avoid permission denied error on ```config.json```

```
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "/home/$USER/.docker" -R
```

## Test if the installation works

```
docker run hello world
```

## Find hello-world image

```
docker image ls
```

## Show all images (the default hides intermediate images):

```
docker image ls --all
```

or:

```
docker image ls -a
```

## Show images in quiet mode (only show numeric ids):

```
docker image ls -q
```

## Show all images in quiet mode:

```
docker image ls -aq
```

## Define a new container with a ```Dockerfile``` 

In this step we define a new container, build it, run it, and share it on Docker public registry.

### The ```Dockerfile```

```Dockerfile
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

### What is in ```requirements.txt```?

```
Flask
Redis
```

### The app: ```app.py```

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

### Build this app

The option ```-t``` will apply the friendly name.

```
docker build -t friendlyhello
```

### Check the built app in the local Docker image registry

```
docker image ls
```

### Run the app
The local machine's port 4000 is mapped to the container's published port 80 using ```-p```:

```
docker run -p 4000:80 friendlyhello
```

The URL where the app can be checked is

```
http://localhost:4000
```

Check the app from the terminal using ```curl```:

```
curl http://localhost:4000
```

### Run the app in the background in detached mode

```
docker run -d -p 4000:80 friendlyhello
```

Check the Docker container:

```
docker container ls
```

### Share the image 
This step needs Docker ID - sign up at ```https://cloud.docker.com```.

#### Login to the Docker public registry:

```
docker login
```

#### Tagging the image

Replace ```<username>```, ```<repository>``` and ```<tag>``` with your username, repository name and tag.

```
docker tag friendlyhello <username>/<repository>:<tag>
```

#### Publish the image

```
docker push <username>/<repository>:<tag>
```

The image can be seen on ```https://hub.docker.com/```.

#### Run the published image on any machine

```
docker run -p 4000:80 <username>/<repository>:<tag>
```

## List all containers, also those not running

```
docker container ls -a
```

## Gracefully stop a container
```
docker container stop <hash>
```

## Kill a container - force shutdown
```
docker container kill <hash>
```

## Remove a container 
```
docker container rm <hash>
```

## Remove all containers
```
docker container rm $(docker container ls -a -q)
```

## List all images on this machine
```
docker container ls
```

## Remove an image from the machine
```
docker container rm <image id>
```

## Remove all images from this machine
```
docker image rm $(docker image ls -a -q)
```

## Login to Docker public registry
```
docker login
```

## Tag image for upload to registry
```
docker tag <image> <username>/<repository>:<tag>
```

## Upload tagged image to registry
```
docker push <username>/<repository>:<tag>
```

## Run image from registry
```
docker run <username>/<repository>:<tag>
```

## Building a service with ```docker-compose.yml```

In this part we will scale the app and enable load balancing.

### Create YAML file: ```docker-compose.yml```

```yaml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

### Use swarm manager to init swarm cluster
```
docker swarm init
```

### Deploy the service

The app name is ```getstartedlab```

```
docker stack deploy -c docker-compose.yml getstartedlab
```

This will run our service stack with 5 container instances of our deployed image on one host.

### Get service id

```
docker service ls
```

The service name is ```getstartedlab_web```.

### List the tasks of the service

A single container running in a service is a task.

```
docker service ps getstartedlab_web
```

### List containers - tasks are also shown
```
docker container ls 
```

### Call the app
```
curl http://localhost:4000
```

If this is run several times, the hostname, which contains the container id, will change. This demonstrates the load balancing.

By changing the ```replicas``` value in ```docker-compose.yml``` the app can be scaled. The ```docker stack depliy -c docker-compose.yml getstartedlab``` command needs to be re-run after the change. An in-place update will happen, no need to shut down the app.

### Take down the app

```
docker stack rm getstartedlab
```

### Take down the swarm
```
docker swarm leave --force
```

### Inspect the task or the container
```
docker inspect <task or container id>
```

## Swarms

Applications can be deployed onto a cluster, running on multiple machines. Multiple machines can be joined into a cluster which cluster is called a "swarm". (So from this point the "cluster" and the "swarm" will be used interchangeably.) Machines in a swarm can be physical or virtual, and after joining the cluster, they are called nodes. So a swarm is a group of machines running Docker and joined into a cluster. After building of this swarm, the usual Docker commands can be used, but they will be executed by a swarm manager. 

The swarm manager can follow strategies for executing the commands, for example:
* emptiest node: the least utilized machines will be filled with containers
* global: each machine gets exactly one instance of the specified container

Roles of the machines in a swarm:
* swarm manager: 
  * execute commands
  * authorizes machines to join the swarm
* worker:
  * provide capacity for the work (they cannot tell other machines what to do)

Docker can be used in:
* single-host mode: no swarm, cointainers are run in the single host
* swarm mode: enables use of swarm. The current machine will become instantly a swarm manager.

### Set up swarms

### Deploy app on the swarm cluster

### Cleanup and reboot


