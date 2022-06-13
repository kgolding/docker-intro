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

1. `docker run caddy` will run the container, but we have now way to communicate with it! _Use `Ctrl-C` to exit the docker container._
    * In this command `caddy` is referring to a docker image. It is made up of layers, and docker will automatically download the image/layers as needed and these are keep locally to make future runs much quicker.
1. `docker run -p 8080:80 caddy` Now we can access port 80. _Use `Ctrl-C` to exit the docker container._
1. Lets create web page `echo "Hello world" > www/index.html`
1. `docker run -p 8080:80 -v "$PWD/www/index.html":/usr/share/caddy/index.html caddy` Now we can see our index.html file at http://localhost:8080/ _Use `Ctrl-C` to exit the docker container._
1. `docker run -p 8080:80 -v "$PWD/www":/usr/share/caddy caddy` Now, any files in www can be accessed
1. Create a second web page `echo "Hello world 2" > www/index2.html` _Use `Ctrl-C` to exit the docker container._
1. Now lets run the docker container in the background by ading the `-d` switch `docker run -d -p 8080:80 -v "$PWD/www":/usr/share/caddy caddy`
1. View the running containers with `docker container ls`
1. View the all containers with `docker container ls -a` and tidy them up using `docker container prune`
1. Stream the containers log `docker container logs -f <id/name>`

## DEMO 2: Compiling go code

Here's how to compile some golang source code to a executable file without installing go

1. `docker run --rm -v "$PWD":/app -w /app golang:1.13 go build -v`

We are using the `golang` official image with the tag `1.13`. If the tag is not specified then you'll get the `latest` tag by default.

https://hub.docker.com/_/golang

And there are similar images available for most languages!

## Demo 3: Interacting with a container

`docker run -it alpine sh` will run an **alpine** linux container allowing us to interact in a terminal (`-it`) and will run the `sh` (aka shell) command.

To run a command within an existing/running container use `docker exec -it <id/name> <command>`

# Container names

When a container starts, it is given an ID and a random name such as "agitated_mahavira", but we can define a name of our choice using the `--name` option. e.g. `docker run --name MyContainerName caddy`

Containers can be referred to by name, full ID or short ID as displayed in the `docker container ls` output.

# Volumes

When an image is built, the author can choose to expose any number of directories as volumes, and these are used to persist data when a container stops. By default a container has no persistent storage!

From our Caddy example above, we could add a volume by adding a `-v` option which will create a `caddy_data` volume. In this example `caddy_data` is the given name of the volume and `/data` is the path the image author defined to hold caddy's data.

> `docker run --rm -d -p 8080:80 --name mycaddy -v caddy_data:/data caddy`

To find out where the actual files are being stored, run `docker inspect -f '{{ json .Mounts }}' mycaddy | jq`

# Docker-compose

`docker-compose` lets us run multiple containers from a single configuration file.

@TODO Example?

## Mini docker cheat sheet

### Containers
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

### Images
* `docker image ls" to list local images (add `-a` to see intermediate images)
* `docker image prune` to delete dangling (unused) images

### Volumes
* `docker inspect -f '{{ json .Mounts }}' <container name, full or short id> | jq` view a containers volumes
