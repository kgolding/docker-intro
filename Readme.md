# Red-Sprite Docker introduction
Kevin Golding - 14th June 2022

# What is docker

Docker is a set of tools for building, running and managing containers aka lightweight virtual machines.

# Why

It lets you spin up a new virtual machine from a massive range of 'templates' without worrying about installing dependencies or library versions as each container has all of it's required files/libraries etc. The docker container runs in isolation (although you can let it access local files if you want too).

# Prerequisites

You can either:

 * Install docker locally
 * Use https://labs.play-with-docker.com/ (requires free login)

# Live demo's

## DEMO 1: Caddy the web server

**Caddy** is web server, which we'll use in a very simple way to host some files.

https://hub.docker.com/_/caddy

1. `docker run caddy` will run the container, but we have now way to communicate with it!
    * In this command `caddy` is referring to a docker image. It is made up of layers, and docker will automatically download the image/layers as needed and these are keep locally to make future runs much quicker.
    * _Use `Ctrl-C` to exit the docker container._
1. `docker run -p 8080:80 caddy` Now we can access port 80 in the container via 8080 on the host e.g. http://localhost:8080/
    * _Use `Ctrl-C` to exit the docker container._
3. Lets create web page `mkdir www` and `echo "Hello world" > www/index.html`
4. `docker run -p 8080:80 -v "$PWD/www/index.html":/usr/share/caddy/index.html caddy` Here we bind/mount a local file into the container, and now we can see our index.html file at http://localhost:8080/
    * _Use `Ctrl-C` to exit the docker container._
7. Create a second web page `echo "Hello world 2" > www/index2.html`
6. `docker run -p 8080:80 -v "$PWD/www":/usr/share/caddy caddy` This time we bind a directory instead, and any files in www can be accessed by the container e.g. http://localhost:8080/index2.html
    * _Use `Ctrl-C` to exit the docker container._
9. Now lets run the docker container in the background by ading the `-d` switch `docker run -d -p 8080:80 -v "$PWD/www":/usr/share/caddy caddy`
10. View the running containers with `docker container ls`
11. View the all containers with `docker container ls -a` and tidy them up using `docker container prune`
12. Stream the containers log `docker container logs -f <id/name>`
13. Stop the container using `docker container stop <id/name>`

## DEMO 2: Compiling and running go code

Here's how to compile some golang source code to a executable file without installing go. There's a super simple go application source code in `main.go` to test with.

> `docker run --rm -v "$PWD":/app -w /app golang:1.13 go run -v helloworld.go`

We are using the `golang` official image with the tag `1.13`. If the tag is not specified then you'll get the `latest` tag by default.

https://hub.docker.com/_/golang

And there are similar images available for most languages!

## Demo 3: Interacting with a container

Run an **alpine** Linux container allowing us to interact in a terminal (`-it`) and run the `sh` (aka shell) command on startup:

> `docker run -it alpine sh`

To run a command within an existing/running container use `docker exec -it <id/name> <command>`

## Demo 4: Building our own image

You build your own images using a `Dockerfile` such as:

```
FROM alpine
RUN apk add micro-tetris
CMD ["tetris"]
```

To build the local image, including tagging it `microtetris` (aka the image name):

> `docker build -t microtetris .`

We can now run a container using the new image:

> `docker run -it microtetris`

Local images can be pushed into a docker registry and shared with the world!

Each command in a `Dockerfile` creates a new layer, and the layers are cached to help speed up future builds.

# Container names

When a container starts, it is given an ID and a random name such as "agitated_mahavira", but we can define a name of our choice using the `--name` option. e.g. `docker run --name mycaddy caddy`

Containers can be referred to by name, full ID or short ID as displayed in the `docker container ls` output.

# Volumes

When an image is built, the author can choose to expose any number of directories as volumes, and these are used to persist data when a container stops. By default a container has no persistent storage!

From our Caddy example above, we could add a volume by adding a `-v` option which will create a `caddy_data` volume. In this example `caddy_data` is the given name of the volume and `/data` is the path the image author defined to hold caddy's data.

> `docker run --rm -d -p 8080:80 --name mycaddy -v caddy_data:/data caddy`

To find out where the actual files are being stored, run `docker inspect -f '{{ json .Mounts }}' mycaddy | jq`

# Simple big picture summary _so far_

* A `Dockerfile` is used to build...
    * An `image` which is made of `layers` (created by each `Dockerfile` instruction), and when run it creates a...
        * `Container` which has:
            * `id`
            * `name`
            * `volume` (zero or more)
                * Which can be `mounted` to persist storage
            * `ports` that can be exposed
            * and more
* `Docker hub` is a registry of images (you can host you own using https://hub.docker.com/_/registry)

---

# Docker-compose

`docker-compose` lets us run multiple containers from a single configuration file.

This example dockerfile runs both a mysql database container, and a phpmyadmin container that connects to the mysql container using dockers build-in dns.

```
version: '3'
 
services:
  db:
    image: mysql:5.7
    container_name: db
    environment:
      MYSQL_ROOT_PASSWORD: my_secret_password
      MYSQL_DATABASE: app_db
      MYSQL_USER: db_user
      MYSQL_PASSWORD: db_user_pass
    volumes:
      - dbdata:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: pma
    links:
      - db
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_ARBITRARY: 0
      PMA_USER: db_user
      PMA_PASSWORD: db_user_pass
    ports:
      - 8081:80
volumes:
  dbdata:
```

Run using the following command in the same directory as the docker-compose.yml file:

> `docker-compose up`

Now open http://localhost:8081/

Notes:
* Port 8081 is mapped to port 80 on the phpmyadmin container
* The mysql data will be persisted in a `dbdata` volume
* To directly connect to the mysql server from the host you would need to add a ports entry in the db services section

---

# Mini docker cheat sheet

## Containers
* `docker run [options] <image name> [command]`
  * `-d` run in the background, use `docker container ls` to see running containers
  * `-p <external port>:<container port>` to allow access/open/map a containers port
  * `-v <local file/directory>:<container file/directory>` to bind a local file/directory into the container. Paths must be absolute and you can use $PWD for the current directory
  * `--rm` removes the container when it stops
  * `--restart unless-stopped` to restart the container automatically (options are `no`/`on-failure`/`always`/`unless-stopped`)
  * `-it` to interact with the container often used when providing a [command]
* `docker container ls` view running containers, add `-a` to see all containers
* `docker container logs -f <id/name>` to stream a given containers logs
* `docker container stop <id/name>` to stop containers
* `docker exec -it <id/name> <command>` run a command in an existing container
* `docker container prune` to delete unused (not running) containers

## Images
* `docker image ls` to list local images (add `-a` to see intermediate images)
* `docker image prune` to delete dangling (unused) images
* `docker build -t <name> .` to build an image from a `Dockerfile` in the same directory

## Volumes
* `docker inspect -f '{{ json .Mounts }}' <id/name> | jq` view a containers volumes

## Docker-compose
* `docker-compose up` runs the docker-compose.yml file in the same directory, add `-d` to run in the background
* `docker-compose down` runs the docker-compose.yml file in the same directory
* `docker-compose logs` displays the logs, add `-f` to stream the logs


