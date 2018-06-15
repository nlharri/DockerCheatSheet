# Cheat Sheet for Docker

## Test Docker version

```
docker --version
```

## 2. View more details

```
docker info
```

or: 

```
docker version
```

## 3. Avoid using ```sudo``` when running Docker commands

### 3.1. Create docker group

```
sudo groupadd docker
```

### 3.2. Add user to docker group

```
sudo usermod -aG docker $user
```

(After this it is necessary to logout and login again.)

### 3.3. Test if docker commands can be run without sudo

```
docker run hello-world
```

### 3.4. (optional) Avoid permission denied error on config.json

```
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "/home/$USER/.docker" -R
```

## 3. Test if the installation works

```
docker run hello world
```

## 4. Find hello-world image

```
docker image ls
```
