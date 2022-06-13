# Red-Sprite Docker introduction
Kevin Golding - 14th June 2022

# What is docker

Docker is a set of tools for building, running and managing containers aka lightweight virtual machines.

# Why

It lets you spin up a new virtual machine from a massive range of 'templates' without worrying about installing dependencies or library versions as each container has all of it's required files/libraries etc. The docker container runs in an isolation (although you can let it access local files if you want too).

# Prerequisites

You can either:

 * Install docker locally
 * Use https://labs.play-with-docker.com/ (requires login)

# Live demo's

## DEMO: Caddy the web server

`Caddy` is web server, which we'll use in a simple mode to host an html file.

https://hub.docker.com/_/caddy

### Running caddy

1. `docker run caddy` will run the container, but we have now way to communicate with it!
    * `Caddy` is a docker image. It is made up of layers, and docker will automatically download the image/layers as needed. These are keep locally to make future runs much quicker.
1. `docker run -p 8080:80 caddy` Now we can access port 80
1. Lets create web page `echo "Hello world" > www/index.html`
1. `docker run -p 8080:80 -v "$PWD/www/index.html":/usr/share/caddy/index.html caddy` Now we can see our index.html file
1. `docker run -p 8080:80 -v "$PWD/www":/usr/share/caddy caddy` Now, any files in www can be accessed
1. Create a second web page `echo "Hello world 2" > www/index2.html`
1. Now lets run the docker container in the background by ading the `-d` switch `docker run -d -p 8080:80 -v "$PWD/www":/usr/share/caddy caddy`
1. View the running containers with `docker container ls`
1. View the all containers with `docker container ls -a` and tidy them up using `docker container prune`
1. Stream the containers log `docker container logs -f <container name, full or short id>`

## DEMO: Compiling go code

Here's how to compile some golang source code to a executable file without installing go

1. `docker run --rm -v "$PWD":/app -w /app golang:1.13 go build -v`

We are using the `golang` official image with the tag `1.13`. If the tag is not specified then you'll get the `latest` tag by default.

https://hub.docker.com/_/golang

And there are similar images available for most languages!

## Mini docker cheat sheet

### Containers
* `docker run [options] <image name>`
  * `-d` run in the background, use `docker container ls` to see running containers
  * `-p <external port>:<container port>` to allow access/open/map a containers port
  * `-v <local file/directory>:<container file/directory>` to bind a local file/directory into the container. Paths must be absolute and you can use $PWD for the current directory
  * `--rm` removes the container when it stops
  * `--restart unless-stopped` to restart the container automatically (options are `no`/`on-failure`/`always`/`unless-stopped`)
* `docker container ls` view running containers, add `-a` to see all containers
* `docker container logs -f <container name, full or short id>` to stream a given containers logs
* `docker container stop <container name, full or short id>` to stop containers

### Images
* `docker image ls" to list local images (add `-a` to see intermediate images)
* `docker image prune` to delete dangling (unused) images

## FAQ

* Access the terminal inside the container:
  * Start with a terminal @TODO
  * Get into a running container @TODO

  
