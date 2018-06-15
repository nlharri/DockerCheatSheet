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
docker tag <image> <username>/<repository>:tag
```

## Upload tagged image to registry
```
docker push <username>/<repository>:tag
```

## Run image from registry
```
docker run <username>/<repository>:tag
```
