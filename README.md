# Docker commands

# For containers and images

## For listing containers which are even stopped..

```bash
docker container ls -a
docker ps -a
```

## Deleting containers

```bash
docker container rm -rf container_name
```

## For listing images

```bash
docker image ls
docker images
```

## For running container in detach mode with a port as well as volume mounted

```bash
docker container run --name webserver -d -p 80:80 -v test:/usr/share/nginx/html nginx
```

## For pulling/pushing image and building a image..

```bash
docker image pull busybox
docker image push busybox
docker image build -t hello . (run this where Dockerfile is present)
```

## For creating a tar and loading a tar

```bash
docker save -o hello.tar busybox
docker load -i hello.tar
```

## Docker container lifecycle..

```bash
docker image pull imageName
docker container run imageName
docker container start containerID
docker container stop containerID
docker container pause containerID
docker container unpause containerID
```

# For Docker volumes and network

## For listing volumes and network..

```bash
docker network ls
docker volume ls
```

## For creating a volume..

```bash
docker volume create volumeName
docker container run --name containerName -v volumeName:/path/in/container busybox
```

You can do the above in single line as well..

```bash
docker container run --name webserver -v C:\users\beayu:/app alpine
```

## For creating a network..

- Docker containers work like VMs.
- Every Docker container has network connections
- Docker Network Drivers:
  - None
  - Bridge
  - Host
  - Macvlan
  - Overlay

Docker supports several networking modes, each designed for specific use cases. Here's a brief overview of the differences between `none`, `host`, `bridge`, `macvlan`, and `overlay` network drivers:

1. **None**:

   - **Use Case**: Completely disables networking for the container.
   - **Details**: Containers with the `none` network cannot communicate with any external network or other containers. This mode is useful when you want to completely isolate a container's network stack.

2. **Host**:

   - **Use Case**: Removes network isolation between the container and the Docker host, and uses the host’s networking directly.
   - **Details**: Containers run on the host's network stack, meaning they share the host's IP address and port space. This mode offers the best network performance but the least isolation.

3. **Bridge**:

   - **Use Case**: The default network mode for containers.
   - **Details**: Creates a virtual network within the host where containers get their own IP addresses, similar to VMs. Containers can communicate with each other and the host through this virtual network. Port mapping is used to allow external access.

4. **Macvlan**:

   - **Use Case**: Allows containers to appear as physical devices on the network.
   - **Details**: Assigns a MAC address to containers, making them appear as physical devices on the network. This allows containers to communicate with the network without passing through the Docker host's network stack. Useful for legacy applications expecting to be directly connected to the physical network.

5. **Overlay**:
   - **Use Case**: Networking across multiple Docker hosts, often used in Docker Swarm.
   - **Details**: Creates a distributed network among multiple Docker daemon hosts. This allows swarm services to communicate with each other, even if they’re running on different Docker hosts. Overlay networks require some sort of key-value store like Consul or etcd as a backend for distributed state.

Each of these network drivers serves different needs, from complete network isolation (`none`) to full integration with the host network (`host`), simple inter-container communication (`bridge`), physical network integration (`macvlan`), and multi-host networking (`overlay`).

```bash

docker network create [networkName]
docker network create bridge1
docker container run --name [containerName] --net [networkName] [imageName]
docker container run --name c1 --net bridge1 alpine sh
docker network inspect bridge1
docker container run --name c2 --net bridge1 alpine sh
docker network connect bridge1 c2
docker network inspect bridge1
docker network disconnect bridge1 c2

```

## Creating a new network using customized network parameters:

```bash
docker network create --driver=bridge --subnet=10.10.0.0/16 --ip-range=10.10.10.0/24 --gateway=10.10.10.10 newbridge
```

Docker Network: Host

## Containers reach host network interfaces (--net host)

```bash

docker container run --name [containerName] --net [networkName] [imageName]
docker container run --name c1 --net host alpine sh

```

## Docker Network: MacVlan

Each Container has its own MAC interface (--net macvlan)

```bash

docker network create --driver=macvlan --subnet=10.10.0.0/16 --ip-range=10.10.10.0/24 --gateway=10.10.10.10 newmacvlan
docker container run --name [containerName] --net [networkName] [imageName]
docker container run --name c1 --net macvlan --mac-address=02:00:00:00:00:01 --ip=244.178.44.111 --gateway=38.0.101.76 newmacvlan
```

## Docker log

`docker logs --details containerName`

## Docker Stats/Memory-CPU Limitations

```bash
docker stats
docker container run --name [containerName] --memory=512m --memory-swap=512m --cpu-shares=512 --cpu-period=100000 --cpu-quota=50000 nginx
```

## Docker Environment Variables

docker run -e VARIABLE=value image

## Dockerfile Important

RUN VS CMD VS ENTRYPOINT

## Dockerfile: RUN, CMD, and ENTRYPOINT

- **RUN**: Executes a command in a new layer and creates a new image. Useful for installing software, or performing some other setup task. RUN does not override the default command, but it can be overridden with the docker run --entrypoint flag.

- **CMD**: Sets the default command to run in the container. If the user does not specify a command when starting the container, then the default command defined in CMD will be executed. CMD can be overridden with the docker run command.

- **ENTRYPOINT**: Defines the executable to run when the container starts. Unlike CMD, it does not create a shell and cannot be overridden. The ENTRYPOINT instruction should be used when the container will be run as an executable.

In summary:

- **RUN**: setup, install software, create files
- **CMD**: default command to run in the container
- **ENTRYPOINT**: executable to run when the container starts

Sample Docker Files

```dockerfile

FROM python:alpine3.7
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 5000
CMD python ./index.py

FROM ubuntu:18.04
RUN apt-get update -y
RUN apt-get install default-jre -y
WORKDIR /myapp
COPY /myapp .
CMD ["java","hello"]
```

## Multi-stage Dockerfile (look at AS in FROM & --from in COPY command..)

```dockerfile

FROM mcr.microsoft.com/java/jdk:8-zulu-alpine AS compiler
COPY /myapp /usr/src/myapp
WORKDIR /usr/src/myapp
RUN javac hello.java

FROM mcr.microsoft.com/java/jre:8-zulu-alpine
WORKDIR /myapp
COPY --from=compiler /usr/src/myapp .
CMD ["java", "hello"]
```

## Brief about Docker compose..

```yaml
version: "3.8"

services:
  mydatabase:
    image: mysql:5.7
    restart: always
    volumes:
      - mydata:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    networks:
      - mynet
  mywordpress:
    image: wordpress:latest
    depends_on:
      - mydatabase
    restart: always
    ports:
      - "80:80"
      - "443:443"
    environment:
      WORDPRESS_DB_HOST: mydatabase:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    networks:
      - mynet
volumes:
  mydata: {}
networks:
  mynet:
    driver: bridge
```

After saving the file as "docker-compose.yml", run the following commands where the docker-compose file is, to create containers, volumes, networks:

```bash

docker-compose up -d
docker-compose down

```
