# Docker Mastery: with Kubernetes +Swarm from a Docker Captain

## Quick Start
### What is Docker?
- example Dockerfile
```docker
FROM python #starts with python layer
RUN pip install flask #new layer, installing  flask
WORKDIR /app
COPY .. # copy resources from current catalog into it's own layer
CMD python app.py
# layers together form an image
# docker build reates an image with only an application and the things we need and nothing else
```
- Docker registry - universall app distribution
- Image has a unique SHA hash
- `docker push` pushes images with all it's layers to a registry
- `docker pull` pulls an image form a registry
- `docker run my-python-app` runs container - namespace isolates images
- [Kubernetes vs. Docker](https://www.bretfisher.com/kubernetes-vs-docker/)
- [About the Open Container Initiative](https://opencontainers.org/about/overview/)
- [Lecture on Github](https://github.com/BretFisher/udemy-docker-mastery/blob/main/intro/what-is-docker/what-is-docker.md)
### Quick container run
- [Play with Docker](https://labs.play-with-docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Official Images](https://docs.docker.com/docker-hub/repos/manage/trusted-content/official-images/)
- [Apache httpd image](https://hub.docker.com/_/httpd)
- [Quick Container Run](https://github.com/BretFisher/udemy-docker-mastery/blob/main/intro/quick-container-run/quick-container-run.md)

#### Playing with Docker
- Open 'Play with Docker', login, add a sesion
```
$ docker version

$ docker run -d -p 8800:80 httpd

$ curl localhost:8800
```
### Why Docker?
- [A Brief History of Containers](https://www.aquasec.com/blog/a-brief-history-of-containers-from-1970s-chroot-to-docker-2016/)
- [Why does Docker exist? (Github)](https://github.com/BretFisher/udemy-docker-mastery/blob/main/intro/why-docker/why-docker.md)

## Course Intro

- [Course Materials on Github](https://github.com/bretfisher/udemy-docker-mastery)
- ["Cloud Native DevOps" Discord](https://devops.fan/)
- [YouTube channel](https://www.youtube.com/channel/UC0NErq0RhP51iXx64ZmyVfg)
### Q&A
#### Bind for 0.0.0.0:80 failed: port is already allocated. -OR- port already in use -OR- permission denied.
This will happen if you are attempting to start a new container with a port that is already in-use on your machine. Remember in TCP/UDP, only one application/service can use a single IP+PORT at a time. This doesn’t change with containers when you use `-p`  to bind to the host IP+PORT.

**You'll need to either stop the app that's using that port, or just run your container on a different (available) port.**

First run `docker ps` to check if there are any containers using this port - if there are not; you likely have a non-Docker related application running on your machine that is using this port. Maybe IIS, maybe Apache, etc.

If you are on a Mac, you can check what is using the port with the command: `lsof -i :<port>`  

If you are on Windows, you can check what is using which port with: `netstat`  

Of course - if you don’t have a reason to specifically use the port that is throwing this error, simply run your container on another port. Remember, the syntax is `<host port>:<container port>`  , so binding to port 8888 on your host machine with a container that uses port 80, would look like: `docker container run -p 8888:80 your_image`  

#### What happened to Docker Toolbox?
It was deprecated by Docker in lieu of Docker Desktop, which now works on all editions of Windows 10/11, macOS, and Linux Desktop.  If you want a multi-node setup locally, then Multipass is a good replacement for Toolbox.

#### $(pwd) in Windows is getting an error for bind-mounts: C:\Program Files\Docker Toolbox\docker.exe: invalid reference format.
First, you should no longer be using CMD.exe (Command Prompt) or PowerShell for Docker commands, because it tends to be a lot easier to just run them from a WSL Linux (bash/zsh) command where `$(pwd)` will work. But, if you'd like more info on the "why", read on.

PowerShell has a few minor differences in command format. This is a PowerShell thing, not a docker thing. When using the shell path shortcut "pwd":

For PowerShell use: `${pwd}`

For cmd.exe "Command Prompt use: `%cd%`

bash, sh, zsh, and Docker Toolbox Quickstart Terminal use: `$(pwd)` 

Note, if you have spaces in your path, you'll usually need to quote the whole path in the docker command.

There's another issue sometimes seen, where other apps can mess up your path: https://stackoverflow.com/questions/50608301/docker-mounted-volume-adds-c-to-end-of-windows-path-when-translating-from-linux

#### I hit Ctrl-C in Windows, and the Container is still running

I recommend you use WSL Linux with bash or zsh to avoid this quirky inconsistency. If you'd like to know more, read on:

In Windows, there's a quirk with the built-in Powershell and Command Prompt terminals. They don't interpret ctrl-c the same way as Linux, Unix, and macOS. They won't shutdown the container, and you'll need to use `docker stop` commands.

#### How do I cleanup space (images etc.)?
Run prune commands https://www.udemy.com/docker-mastery/learn/v4/t/lecture/7407918?start=0

#### Bind Mount Won't Show Up In Container

This is usually a Docker for Windows issue, where you need to go into Docker Settings GUI (lower right icon) and uncheck the drive where your code is, then save, and then re-check that drive to re-apply the file sharing permissions between the Linux VM and the Windows OS.

#### Starting container process caused "exec: \"ping\": executable file not found in $PATH": unknown

That error is telling you that ping is not available in the image you’re trying to run it from. Official images have changed over time and the official nginx default image (nginx:latest) no longer has ping in it by default.  Image nginx:alpine should still have ping installed (a few of my videos show utilities like ping that are no longer in those images).

If it's a debian-based image (the default nginx) then you can also use `apt-get update && apt-get install -y iputils-ping`   inside the container to install it.

Lastly, I keep a “bunch of troubleshooting and handy admin utilities” in an image here that you can run ping from: `bretfisher/netshoot`
https://www.udemy.com/docker-mastery/learn/v4/questions/3751216  

#### Starting mysql container and running ps causes "ps: command not found"

Like above, this is the container shell telling you the binary "ps" isn't in your path, and not installed in the container. Docker changed the mysql image after the video was recorded and removed the ps utility. You can add it back in using the apt package manager.

`apt-get update && apt-get install procps`

For more info: https://stackoverflow.com/questions/26982274/ps-command-doesnt-work-in-docker-container

#### How to run two container websites on a single port in Docker or Swarm services
This is a bit more advanced, but common for production Swarms. You'll need a "reverse proxy"

https://www.udemy.com/docker-mastery/learn/v4/questions/3931678
#### Error response from daemon: pull access denied.

Double and triple-check the spelling of the image you are pulling; if you are attempting to pull a publicly hosted image - this error will not occur, but if there is a typo and Docker can’t find the image - it will expect that it is a private image and ask you to login.

Also, there are times when the config.json file gets messed up, so try docker logout && docker login. If all that still causes the same issue, try removing `~/.docker/config.json`  and then pull again.

#### Kubernetes vs. Swarm.
I have a dedicated lecture for this: 

https://www.udemy.com/course/docker-mastery/learn/lecture/15094976

#### Does this help with Docker Certified Associate?
Yes, but it’s not a study guide. Here’s the Lecture with info: https://www.udemy.com/docker-mastery/learn/v4/t/lecture/9485678?start=0

#### Ubuntu Container vs. Ubuntu OS, What's the Difference?
https://www.udemy.com/docker-mastery/learn/v4/questions/5390204

#### How to use volumes in Swarm for databases.
https://www.udemy.com/docker-mastery/learn/v4/questions/2675184

#### How do we do backups in docker?
https://www.udemy.com/docker-mastery/learn/v4/questions/2756448

#### Getting a shell in VM’s that run Docker
Workaround: https://www.udemy.com/docker-mastery/learn/v4/questions/3860412

`docker run -it --rm --privileged --pid=host justincormack/nsenter1`

macOS https://www.bretfisher.com/docker-for-mac-commands-for-getting-into-local-docker-vm/

Docker for Windows https://www.bretfisher.com/getting-a-shell-in-the-docker-for-windows-vm/

Docker Toolbox `docker-machine ssh default`

#### Windows firewalls preventing networking or bind mounts in containers
https://www.udemy.com/docker-mastery/learn/v4/questions/3258290

#### Anti-Virus Blocking file sharing in Windows
https://www.udemy.com/docker-mastery/learn/v4/questions/3442460

#### Are containers more secure than VM’s?
https://www.udemy.com/docker-mastery/learn/v4/questions/4020880

#### I have a network proxy and images won’t build
https://stackoverflow.com/questions/23111631/cannot-download-docker-images-behind-a-proxy/

#### Public vs. Private IP for Swarm advertise-addr and data-path-addr
https://www.udemy.com/docker-mastery/learn/v4/questions/3710518

#### Custom Docker Networks, macvlan and IP setting hardcoding
https://www.udemy.com/docker-mastery/learn/v4/questions/3706540

## Setting up Docker
### Installing the right Docker
- [Open Container Initiative](https://opencontainers.org/)
- [Docker Desktop](https://docs.docker.com/desktop/)
- [Local container runtimes in 2024, tools for running containers for local dev
](https://docs.google.com/spreadsheets/d/1ZT8m4gpvh6xhHYIi4Ui19uHcMpymwFXpTAvd3EcgSm4/edit?gid=0#gid=0)

- 3 major ways to run containers
  - Locally(Docker Desktop, RD)
  - Servers(Docekr Engine, K8s)
  - Paas(Cloud Run, Fargate)
  
### Instaling Docker The Fast way
#### Installing on Windows 10 and Windows 11 (any Edition)
- https://docs.docker.com/desktop/windows/install/

- This is the best experience on Windows, which uses WSL2. If you're new to WSL2, it's the best way to run Linux on Windows, and Docker Desktop will walk you through enabling it. Docker can technically still work on Hyper-V with "Windows Containers," but most of this course focuses on Linux Containers, which WSL2 is great at.

- After installing Docker Desktop for Windows. It is recommended installing a Windows Store Ubuntu for WSL2 distro and using its shell in Windows Terminal is the best CLI experience! Make sure to check your Docker Desktop settings to give it enough resources and enable all your WSL2 distros for Docker Desktop.

#### Installing on Windows 7 or Windows 8
- Docker Desktop don't work in these older versions. You'll need to use Hyper-V or install VirtualBox and manually setup a Linux VM.

#### Installing on Linux Desktop

- https://docs.docker.com/desktop/linux/install/
- Installing on Linux Server https://docs.docker.com/engine/install/

- Do *not* use your built-in default packages  apt/yum install docker.io because those packages are old and not the Official Docker-Built packages. 

- Prefer to use Docker's automated script to add their repository and install all dependencies: curl -sSL https://get.docker.com/ | sh  but you can also install in a more manual method by following specific instructions that Docker provides for your Linux distribution.

#### Use Play With Docker
- https://labs.play-with-docker.com/

- When you don't have local admin, or maybe your machine doesn't have enough resources. The best free option here is to use play-with-docker.com, which will run one or more Docker instances inside your browser, and give you a terminal to use it with. You can create multiple machines on it and even use the URL to share the session with others in a sort of collaborative experience.
### Docker for Windows: Setup and Tips
- [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
  - Use WSL2 instead of Hyper-V
  - Add shortcut to desktop
  - Docker desktop / Configuration / Resources / WSL Integration - check **Enable integration with my default WSL distro**
  - Go to Windows Store and download Ubuntu for WSL
  - [Course GitHub Repo](https://github.com/BretFisher/udemy-docker-mastery)
  
### Docker for Linux Desktop: Setup and Tips
- [Install Docker Desktop on Linux](https://docs.docker.com/desktop/setup/install/linux/)
- [Course GitHub Repo](https://github.com/BretFisher/udemy-docker-mastery)
- Tips
  - point **Resources File Sharing** to `/home`.
  
### Docker for Linux Server: Setup and Tips
- [Install Docker Engine on Linux](https://docs.docker.com/engine/install/)
- Tips:
  - for installing you can also use a script.
```
curl -fsSL https:// get.docker.com -o get-docker.sh
sh get-docker.sh
```
  - command line `apt install docker-ce-cli`
  - connecting docker servers from a commandline: 
    - set DOCKER_HOST `export DOCKER_HOST=ssh://root@143.244.158.151`
    
### VS Code for DevOps, Docker and YAML Editing
- [Docker Extension Pack for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
- [Visual Studio Code Kubernetes Tools](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools)
- [Microsoft Visual Studio Live Share](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare)
- [Visual Studio Code Remote Development Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
## Creating and using containers
### Checking Docker install and config

```
#verified cli can talk to engine
docker version

#most config values of engine
docker info

docker

docker container run

docker run
```
- The newest format `docker <management_command> <subcommand> (options)`

### Starting a Nginx Web Server
- An Image is the application we want to run (bins, libs, source code)
- A Container is an instance of that image running as a process
- Docker's default image "registry" is called Docker Hub
(hub.docker.com)
- `run` start a new container, `start` starts existing stopped one.
```
docker container run --publish 80:80 nginx
# run in the background, terminal is not blocked
docker container run --publish 80:80 --detach nginx

docker container ls

docker container stop 690

# running containers
docker container ls

# all containers
docker container ls -a

# names are generated randomly unless we specify them
docker container run --publish 80:80 --detach --name webhost nginx

docker container ls -a

# logs for webhost container
docker container logs webhost

docker container top

# running processes in our container 
docker container top webhost

docker container --help

docker container ls -a

docker container rm 63f 690 ode

docker container ls

# removing container with 'force' option
docker container rm -f 63f

docker container ls -a
```
### What happens in 'docker container run'
1. Looks for that image locally in image cache, doesn't find anything
2. Then looks in remote image repository
3. Downloads the latest version (nginx:latest by default)
4. Creates new container based on that image and prepares to start
5. Gives it a virtual IP on a private network inside docker engine
6. Opens up port 80 on host and forwads to port 80 in container
7. Starts container by using the CMD in the image Dockerfile

```
docker container run --publish 8080:80 --name webhost -d nginx:1.11 nginx -T
```

### Container VS. VM: It's Just a Process
- [Cgroups, namespaces, and beyond: what are containers made from?](https://www.youtube.com/watch?v=sK5i-N34im8&list=PLBmVKD7o3L8v7Kl_XXh3KaJl9Qw2lyuFl)
- [Getting a Shell in the Docker for Windows Moby VM](https://www.bretfisher.com/blog/getting-a-shell-in-the-docker-for-windows-vm)
```

docker run --name mongo -d mongo

docker ps

# list processes running inside a container
docker top mongo

docker stop mongo

docker ps

docker top mongo

docker start mongo

docker ps

docker top mongo
```
### Assignment Answers: Manage Multiple Containers

```
docker container run -d -p 3306:3306 --name db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql

docker container logs db

docker container run -d --name webserver -p 8080:80 httpd

docker ps

docker container run -d --name proxy -p 80:80 nginx

docker ps

docker container ls

docker container stop

docker ps -a

docker container ls -a

docker container rm

docker ps -a

docker image ls
```
### What's Going On In Containers: CLI Process Monitoring
- `docker container top` - process list in one container
- `docker container inspect` - details of one container config
- `docker container stats` - performance stats for all containers

```
docker container run -d --name nginx nginx

docker container run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql

docker container ls

docker container top mysql

docker container top nginx

docker container inspect mysql

docker container stats --help

docker container stats

docker container ls
```
### Getting a Shell Inside Containers: No Need for SSH
- **Replacing mysql with mariadb**
``docker container run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=true mariadb``
- `docker container run -it` - start new container interactively
- `docker container exec -it`- run additional command in existing
- [Package Management Essentials](https://www.digitalocean.com/community/tutorials/package-management-basics-apt-yum-dnf-pkg)

```
docker container run -help

# running with extra command bash
docker container run -it --name proxy nginx bash

docker container ls

docker container ls -a

docker container run -it --name ubuntu ubuntu

docker container ls

docker container ls -a

docker container start --help

# re-running a container
docker container start -ai ubuntu

docker container exec --help

docker container exec -it mysql bash

docker container ls

docker pull alpine

docker image ls

# gives an error
docker container run -it alpine bash

docker container run -it alpine sh
```
### Docker Networks: Concepts for Private and Public Comms in Containers
- [Format command and log output](https://docs.docker.com/engine/cli/formatting/)
- `docker container port <container>` - which ports are open for a container
- Docker has its own private virtual network "bridge"
- Each virtual network routes through NAT firewall on host IP
- All containers on a virtual network can talk to each other without -p
- Good practice is to create a network for the related applications
```
docker container run -p 80:80 --name webhost -d nginx
# shows the port mapping
docker container port webhost
# checking the json node
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' webhost
```

### Change In Official Nginx Image Removes Ping
- A change in the official nginx image https://hub.docker.com/_/nginx (nginx  or nginx:latest ) removes ping
- Use `nginx:alpine` instead of `nginx`
### Docker Networks: CLI Management of Virtual Networks
- `docker network ls` - show networks
- `docker network inspect` - inspect a network
- `docker network create --driver` - create a network
- `docker network connect` - attach a network to container
- `docker network disconnect` - detach a network from container
- `--network bridge` - default Docker virtual network, which is NATed behind the  Host IP 
- `--network none` - removes eth0 and only leaves you with localhost interface in container

```
docker network ls

docker network inspect bridge

docker network ls

# creating a new network
docker network create my_app_net

docker network ls

docker network create --help

# running a container with a new network my_app_net
docker container run -d --name new_nginx --network my_app_net nginx:alpine

docker network inspect my_app_net

docker network --help

# Dynamically creates a NIC in a container on an existing virtual network
# <NETWORK_ID> <CONTAINER_ID>
docker network connect becffb3eef1c f3d08cadc1e0
	                                 			   
docker container inspect

docker network disconnect becffb3eef1c f3d08cadc1e0

docker container inspect
```
#### Docker Networks Default Security

- Create your apps so frontend/backend sit on the same Docker network
- Their inter-communication never leaves host
- All externally exposed ports closed by default
- You must manually expose via -p, which is better default security
- This get even better later with Swarm and Overlay networks

### Docker Networks: DNS and How Containers Find Each Other
-  Docker daemon has a built-in DNS server that containers use by default
- Docker defaults the hostname to the container's name, but you can also set the aliases
```
docker container ls

docker network inspect

docker container run -d --name my_nginx --network my_app_net nginx:alpine

docker container inspect my_app_net

docker container exec -it my_nginx ping new_nginx

docker container exec -it new_nginx ping my_nginx

docker network ls

docker container create --help
```
#### Docker Networks: DNS
- Containers shouldn't rely on IP's for inte-cmmunication
- DNS for friendly names is built-in if you use custom networks
- You're using custom networks right?
- This gets way easier with Docker Compose in future Section 


#### Assignment Answers: Using Containers for CLI Testing
- curl update
  - ubuntu: `apt-get update && apt-get install curl` 
  - centos: `yum update curl`
- checking a version: `curl --version`
- Assignment:
  - Use different Linux distro containers to check curl cli tool version
  - Use two different terminal windows to start bash in both centos:7
and ubuntu:14.04, using -it
  - Learn the docker container run —rm option so you can save
cleanup
  - Ensure curl is installed and on latest version for that distro
  - ubuntu: apt-get update && apt-get install curl
    - centos: yum update curl
    - Check curl --version
```
docker container run --rm -it centos:7 bash

docker ps -a

docker container run --rm -it ubuntu:14.04 bash

docker ps -a
```
#### Assignment Answers: DNS Round Robin Testing
- Elasticsearch newer versions require environment variable changes to work correctly. Older elasticsearch versions still work, but don't support Arm64 (Apple Silicon, etc).Alternatives:
  - Use an updated Elasticsearch image and limit its Java memory
  - Replace Elasticsearch with httpenv, a simple HTTP serve
- centos is no longer a supported image by Red Hat, so we can use rockylinux/rockylinux:10 as a drop-in replacement for centos.
- [Round-robin_DNS](https://en.wikipedia.org/wiki/Round-robin_DNS)
- Assignment:
  - Ever since Docker Engine 1.11, we can have multiple containers
on a created network respond to the same DNS address
  - Create a new virtual network (default bridge driver)
  - Create two containers from elasticsearch:2 image
  - Research and use —network-alias search when creating them
to give them an additional DNS name to respond to
  - Run alpine nslookup search with --net to see the two
containers list for the same DNS name
  - Run centos curl -s search:9200 with --net multiple times
until you see both "name" fields show

```
docker network create dude

# --net - network
docker container run -d --net dude --net-alias search elasticsearch:2

docker container ls

docker container run --rm -- net dude alpine nslookup search

docker container run --rm --net dude centos curl -s search:9200

docker container ls

docker container rm -f
```
##### --net-alias vs --link
- With --net-alias, one container can access the other container only if they are on the same network. In other words, in addition to --net-alias foo and --net-alias bar, you need to start both containers with --net foobar_net after creating the network with docker network create foobar_net.
- With --net-alias foo, all containers in the same network can reach the container by using its alias foo. With --link, only the linked container can reach the container by using the name foo.

## Container images. Where to find them and how to use them.
- App binaries and dependencies
- Metadata about the image data and how to run the image
- Not a complete OS. No kernel, kernel modules (e.g. driver)
- Small as one file (your app binary) like a golang static binary
- Big as a Ubuntu distro with apt and Apache, PHP and more installed

### The Mighty Hub: Using Docker Hub Registry Images

- [Docker Hub](http://hub.docker.com) 
- [List of official images](https://github.com/docker-library/official-images/tree/master/library)
- Images are tagged
- Alwayes specify an exact version of image on production

```
# list of images
docker image ls

# downloads the latest
docker pull nginx

docker pull nginx:1.11.9

docker pull nginx:1.11

docker pull nginx:1.11.9-alpine

docker image ls
```

### Images and Their Layers: Discover the Image Cache
- [storage, drivers](https://docs.docker.com/engine/storage/drivers/)
- Union file system (layers)
- Starts with an empty layer (scratch)
- Each change is an another layer
- Each layer has a unique SHA
- We never store and entire stack of image layers more than once if those layers ar ethe same
- When we start a container, docker creates another read/write layer for that container on top the image
- Image in readonly
- If we run a container and change something in a main image, the change is stored in a top (container's) layer. Main image stays unchanged. 
- Layers are not always images. They are often just layers in a image and they do not have their own SHA.

```
docker image ls

# shows layers of changes made in image
docker history nginx:latest

docker history mysql

# JSON metadata about an image
docker image inspect nginx
```

### Image Tagging and Pushing to Docker Hub
- REPOSITORY - usually a username or organisation / repository
- official repositories - they live at the root namespace of the registry, so they do not need account name in front of repo name
- `docker login <server>` - defaults to logging in Hub, but you can override by adding server url

```
docker image tag --help

docker pull mysql/mysql-server

docker image ls

# repository:tag
docker pull nginx:mainline

docker image ls

# changes repository name to bretfisher/nginx, tag will be 'latest' by default
docker image tag nginx bretfisher/nginx

docker image tag --help

docker image ls

docker image push bretfisher/nginx

docker --help

docker login

# local config
cat .docker/config.json

docker image push bretfisher/nginx

# adding a tag 'testing'
docker image tag bretfisher/nginx bretfisher/nginx:testing

docker image push bretfisher/nginx bretfisher/nginx:testing

docker image ls

docker image push bretfisher/nginx:testing

docker image ls
```


### Building Images: The Dockerfile Basics
- https://github.com/BretFisher
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- `FROM` base image
- `ENV` environment variables 
- `&&` chaining commands to reduce no of layers
- linking logs
```   
# forward request and error logs to docker log collector
&& ln -sf /dev/stdout /var/log/nginx/access.log \
&& ln -sf /dev/stderr /var/log/nginx/error.log \
```
- `EXPOSE 80` exposing port which then can be published `-p` 
- `CMD` final command which is executed every time when a container is started

```
cd dockerfile-sample-1

vim Dockerfile
```

```
#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "update.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#
FROM debian:bookworm-slim

LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

ENV NGINX_VERSION   1.29.0
ENV NJS_VERSION     0.9.0
ENV NJS_RELEASE     1~bookworm
ENV PKG_RELEASE     1~bookworm
ENV DYNPKG_RELEASE  1~bookworm

RUN set -x \
    # create nginx user/group first, to be consistent throughout docker variants
    && groupadd --system --gid 101 nginx \
    && useradd --system --gid nginx --no-create-home --home /nonexistent --comment "nginx user" --shell /bin/false --uid 101 nginx \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y gnupg1 ca-certificates \
    && \
    NGINX_GPGKEYS="573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62 8540A6F18833A80E9C1653A42FD21310B49F6B46 9E9BE90EACBCDE69FE9B204CBCDCD8A38D88A2B3"; \
    NGINX_GPGKEY_PATH=/etc/apt/keyrings/nginx-archive-keyring.gpg; \
    export GNUPGHOME="$(mktemp -d)"; \
    found=''; \
    for NGINX_GPGKEY in $NGINX_GPGKEYS; do \
    for server in \
    hkp://keyserver.ubuntu.com:80 \
    pgp.mit.edu \
    ; do \
    echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
    gpg1 --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
    done; \
    test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
    done; \
    gpg1 --export $NGINX_GPGKEYS > "$NGINX_GPGKEY_PATH" ; \
    rm -rf "$GNUPGHOME"; \
    apt-get remove --purge --auto-remove -y gnupg1 && rm -rf /var/lib/apt/lists/* \
    && dpkgArch="$(dpkg --print-architecture)" \
    && nginxPackages=" \
    nginx=${NGINX_VERSION}-${PKG_RELEASE} \
    nginx-module-xslt=${NGINX_VERSION}-${DYNPKG_RELEASE} \
    nginx-module-geoip=${NGINX_VERSION}-${DYNPKG_RELEASE} \
    nginx-module-image-filter=${NGINX_VERSION}-${DYNPKG_RELEASE} \
    nginx-module-njs=${NGINX_VERSION}+${NJS_VERSION}-${NJS_RELEASE} \
    " \
    && case "$dpkgArch" in \
    amd64|arm64) \
    # arches officialy built by upstream
    echo "deb [signed-by=$NGINX_GPGKEY_PATH] https://nginx.org/packages/mainline/debian/ bookworm nginx" >> /etc/apt/sources.list.d/nginx.list \
    && apt-get update \
    ;; \
    *) \
    # we're on an architecture upstream doesn't officially build for
    # let's build binaries from the published packaging sources
    # new directory for storing sources and .deb files
    tempDir="$(mktemp -d)" \
    && chmod 777 "$tempDir" \
    # (777 to ensure APT's "_apt" user can access it too)
    \
    # save list of currently-installed packages so build dependencies can be cleanly removed later
    && savedAptMark="$(apt-mark showmanual)" \
    \
    # build .deb files from upstream's packaging sources
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y \
    curl \
    devscripts \
    equivs \
    git \
    libxml2-utils \
    lsb-release \
    xsltproc \
    && ( \
    cd "$tempDir" \
    && REVISION="${NGINX_VERSION}-${PKG_RELEASE}" \
    && REVISION=${REVISION%~*} \
    && curl -f -L -O https://github.com/nginx/pkg-oss/archive/${REVISION}.tar.gz \
    && PKGOSSCHECKSUM="400593da45fc0195a01138c0c23a06059da1c6a2e26959f2c4c95fbaf63436ff211665ef01392d2b775a0133d5b57680dabe51b840a55f82e89621e84cf651d1 *${REVISION}.tar.gz" \
    && if [ "$(openssl sha512 -r ${REVISION}.tar.gz)" = "$PKGOSSCHECKSUM" ]; then \
    echo "pkg-oss tarball checksum verification succeeded!"; \
    else \
    echo "pkg-oss tarball checksum verification failed!"; \
    exit 1; \
    fi \
    && tar xzvf ${REVISION}.tar.gz \
    && cd pkg-oss-${REVISION} \
    && cd debian \
    && for target in base module-geoip module-image-filter module-njs module-xslt; do \
    make rules-$target; \
    mk-build-deps --install --tool="apt-get -o Debug::pkgProblemResolver=yes --no-install-recommends --yes" \
    debuild-$target/nginx-$NGINX_VERSION/debian/control; \
    done \
    && make base module-geoip module-image-filter module-njs module-xslt \
    ) \
    # we don't remove APT lists here because they get re-downloaded and removed later
    \
    # reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
    # (which is done after we install the built packages so we don't have to redownload any overlapping dependencies)
    && apt-mark showmanual | xargs apt-mark auto > /dev/null \
    && { [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; } \
    \
    # create a temporary local APT repo to install from (so that dependency resolution can be handled by APT, as it should be)
    && ls -lAFh "$tempDir" \
    && ( cd "$tempDir" && dpkg-scanpackages . > Packages ) \
    && grep '^Package: ' "$tempDir/Packages" \
    && echo "deb [ trusted=yes ] file://$tempDir ./" > /etc/apt/sources.list.d/temp.list \
    # work around the following APT issue by using "Acquire::GzipIndexes=false" (overriding "/etc/apt/apt.conf.d/docker-gzip-indexes")
    #   Could not open file /var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages - open (13: Permission denied)
    #   ...
    #   E: Failed to fetch store:/var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages  Could not open file /var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages - open (13: Permission denied)
    && apt-get -o Acquire::GzipIndexes=false update \
    ;; \
    esac \
    \
    && apt-get install --no-install-recommends --no-install-suggests -y \
    $nginxPackages \
    gettext-base \
    curl \
    && apt-get remove --purge --auto-remove -y && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list \
    \
    # if we have leftovers from building, let's purge them (including extra, unnecessary build deps)
    && if [ -n "$tempDir" ]; then \
    apt-get purge -y --auto-remove \
    && rm -rf "$tempDir" /etc/apt/sources.list.d/temp.list; \
    fi \
    # forward request and error logs to docker log collector
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    # create a docker-entrypoint.d directory
    && mkdir /docker-entrypoint.d

# COPY docker-entrypoint.sh /
# COPY 10-listen-on-ipv6-by-default.sh /docker-entrypoint.d
# COPY 15-local-resolvers.envsh /docker-entrypoint.d
# COPY 20-envsubst-on-templates.sh /docker-entrypoint.d
# COPY 30-tune-worker-processes.sh /docker-entrypoint.d
# ENTRYPOINT ["/docker-entrypoint.sh"]
#
EXPOSE 80

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]

```

### Building Images: Running Docker Builds

```
# building an image
docker image build -t customnginx .

docker image ls

docker image build -t customnginx .
```


### Building Images: Extending Official Images
- `WORKDIR /usr/share/nginx/html` equivalent of directory change
- `COPY index.html index.html` copy your source code to container images


```
cd dockerfile-sample-2

vim Dockerfile

docker container run -p 80:80 --rm nginx

docker image build -t nginx-with-html .

docker container run -p 80:80 --rm nginx-with-html

docker image ls

docker image tag --help

docker image tag nginx-with-html:latest bretfisher/nginx-with-html:latest

docker image ls

docker push
```
- Dockerfile

```
# this shows how we can extend/change an existing official image from Docker Hub

FROM nginx:latest
# highly recommend you always pin versions for anything beyond dev/learn

WORKDIR /usr/share/nginx/html
# change working directory to root of nginx webhost
# using WORKDIR is preferred to using 'RUN cd /some/path'

COPY index.html index.html

# I don't have to specify EXPOSE or CMD because they're in my FROM
```

### Assignment Answers: Build Your Own Dockerfile and Run Containers From It
#### Assignment: Build Your Own Image
Dockerfiles are part process workflow and part art
- Take existing Node.js app and Dockerize it
- Make Dockerfile. Build it. Test it. Push it. (rm it). Run it.
- Expect this to be iterative. Rarely do I get it right the first time.
- Details in dockerfile-assignment-1/Dockerfile
- Use the Alpine version of the official 'node' 6.x image
- Expected result is web site at http://localhost
- Tag and push to your Docker Hub account (free)
- Remove your image from local cache, run again from Hub

```Dockerfile
FROM node:6-alpine

EXPOSE 3000

RUN apk add --update tini

RUN mkdir -p /usr/src/app

WORKDIR /usr/src/app
 
COPY package.json package.json
 
RUN npm install && npm cache clean --force
  
COPY . .

CMD [ "tini", "--","node", "./bin/www" ]
```

```
cd dockerfile-assignment-1

vim Dockerfile

docker build -t testnode .

docker container run --rm -p 80:3000 testnode

docker images

docker tag --help

docker tag testnode bretfisher/testing-node

docker push --help

docker push bretfisher/testing-node

docker image ls

docker image rm bretfisher/testing-node
docker container run --rm -p 80:3000 bretfisher/testing-node

```
### Using Prune to Keep Your Docker System Clean (YouTube)
You can use "prune" commands to clean up images, volumes, build cache, and containers. Examples include:

- `docker image prune` to clean up just "dangling" images

- `docker system prune` will clean up everything you're not currently using

- The big one is usually `docker image prune -a` which will remove all images you're not using. Use `docker system df` to see space usage.

Remember each one of those commands has options you can learn with --help.

Here's a YouTube video I made about it: https://youtu.be/_4QzP7uwtvI

Lastly, realize that the Linux VM will *eventually* auto-shrink. You may not see the free space on your host OS right away, and it may take Docker a restart and some idle time before it completes a VM shrink.

## Container Lifetime & Persistent Data: Volumes, Volumes, Volumes
- [Storage](https://docs.docker.com/engine/storage/)
- [12 Fractured Apps](https://medium.com/@kelseyhightower/12-fractured-apps-1080c73d481c#.cjvkgw4b3)
- [12factorapp](https://12factor.net/)
- Containers are usually immutable and ephemeral
- "immutable infrastructure": only re-deploy containers, never change
- Volumes and Bind Mounts
- Volumes: make special location outside of container UFS
- Bind Mounts: link container path to host path

### Persistent Data: Data Volumes
- VOLUME command in Dockerfile
- Also override with docker run -v /path/in/container
- Bypasses Union File System and stores in alt location on host
- Includes it's own management commands under docker volume
- Connect to none, one, or multiple containers at once
- Not subject to commit, save, or export commands
- By default they only have a unique ID, but you can assign name
```

docker pull mariadb

docker image inspect mariadb

docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True mariadb

docker container ls

docker container inspect mysql

docker volume ls

docker volume inspect TAB COMPLETION

docker container run -d --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=True mariadb

docker volume ls

docker container stop mariadb

docker container stop mysql2

docker container ls

docker container ls -a

docker volume ls

docker container rm mariadb mysql2

docker volume ls

# named volume
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mariadb

docker volume ls

docker volume inspect mysql-db

docker container rm -f mysql

docker container run -d --name mysql3 -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mariadb

docker volume ls

docker container inspect mysql3

# when we want to specify a driver
docker volume create --help
```


```json
 "Mounts": [
            {
                "Type": "volume",
                "Name": "27809093b25e2a86b1f1e889e96d164aae3723409990d2166e448d791f858c50",
                "Source": "/var/lib/docker/volumes/27809093b25e2a86b1f1e889e96d164aae3723409990d2166e448d791f858c50/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```


#### Shell Differences for Path Expansion

In the next lecture, you'll learn how to share files and directories between a host and a Docker container. One of the parts of the command line you'll need to type is the host file path you want to share.

With Docker CLI, you can always use a full file path on any OS, but often you'll see me and others use a "parameter expansion" like $(pwd) which means "print working directory".

Here's the important part. Each shell may do this differently, so here's a cheat sheet for which OS and Shell your using. I'll be using `$(pwd)` on a Mac, but yours may be different!

This isn't a Docker thing, it's a Shell thing.

For PowerShell use: `${pwd}` 

For cmd.exe "Command Prompt use: %cd%

Linux/macOS bash, sh, zsh, and Windows Docker Toolbox Quickstart Terminal use: `$(pwd)` 

Note, if you have spaces in your path, you'll usually need to quote the whole path in the docker command.


### Persistent Data: Bind Mounting
- Maps a host file or directory to a container file or directory
- Basically just two locations pointing to the same file(s)
- Skips UFS, and host files overwrite any in container
- Can't use in Dockerfile, must be at container run
- ... run -v /Users/bret/stuff:/path/container (mac/linux)
- ... run -v //c/Users/bret/stuff:/path/container
(windows)
- `$(pwd)` - print out a working directory

```
cd dockerfile-sample-2

pcat Dockerfile

docker container run -d --name nginx -p 80:80 -v $(pwd):/usr/share/nginx/html nginx:alpine

docker container run -d --name nginx2 -p 8080:80 nginx:alpine

docker container exec -it nginx bash  
```


### Assignment Answers: Database Upgrades with Named Volumes
#### Assignment
- Database upgrade with containers
- Create a postgres container with named volume psql-data
using version 9.6.1
- Use Docker Hub to learn VOLUME path and versions needed to run
it
- Check logs, stop container
- Create a new postgres container with same named volume
using 9.6.2
- Check logs to validate
- (this only works with patch versions, most SQL DB's require manual commands to upgrade
DB's to major/minor versions, i.e. it's a DB limitation not a container one)

- Use postgres:15.2


- postgres needs passwords

```
docker volume create psql
docker run -d --name psql1 -e POSTGRES_PASSWORD=mypassword -v psql:/var/lib/postgresql/data postgres:15.1
docker logs psql1
docker stop psql1
docker run -d --name psql2 -e POSTGRES_PASSWORD=mypassword -v psql:/var/lib/postgresql/data postgres:15.2
docker logs psql2
docker stop psql2
```
### Assignment Answers: Edit Code Running In Containers With Bind Mounts

`docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve`

#### Assignment

- Use a [Jekyll](https://jekyllrb.com/) "Static Site Generator" to start a local web server
- Don't have to be web developer: this is example of bridging the gap
between local file access and apps running in containers
- source code is in the course repo under **bindmount-sample-1**
- We edit files with editor on our host using native tools
- Container detects changes with host files and updates web server
- start container with
 `docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyllserve`
- Refresh our browser to see changes
- Change the file in **_posts\** and refresh browser to see changes

## Dockerfile ENTRYPOINT
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
### Buildtime vs Runtime
- Buildtime statements affect the files in the image or how the image is built.
- Runtime statements are typically stored as metadata and affect the container.
- Some statements affect how the image is built and also change container start behavior
- Overwrite statements replace any previous use.
- Additive statements add to any previous use.
- Know your base (FROM) images. Many statement types affect downstream images.
- Understanding these effects helps troubleshoot Dockerfile and container issues.

|Buildtime "docker build"|Both| Runtime "docker run"|
|------------------------|----|---------------------|
|ADD, COPY, FROM, RUN, ONBUILD|**Additive** - These statements add to the image and can't be overwritten FROM with a second statement.  | EXPOSE, VOLUME|
|ARG| **Overwrite** - These statements replace their previous use. For key=value statements, a reused key name replaces the previous value. |STOPSIGNAL, CMD, ENTRYPOINT, HEATHCHECK|

### What is ENTRYPOINT
- ENTRYPOINT executes a command on container start.
- ENTRYPOINT is a **Runtime** statement, stored as metadata with an image.
- Only the last ENTRYPOINT in a Dockerfile is used, making it an **Overwrite** type.
- A container needs at least a CMD or an ENTRYPOINT to know how to start.
- ENTRYPOINT requires more typing to overwrite compared to CMD, so it's rarely used by itself as a replacement for CMD.
- You can overwrite ENTRYPOINT with `docker run --entrypoint "something" <image>`.
- [Entrypoint referene](https://docs.docker.com/reference/dockerfile/#entrypoint)
- [Busybox](https://busybox.net/about.html)

```
> docker run busybox

> docker inspect busybox

    "Cmd": [
                "sh"
            ],
            "Image": "",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,

> docker run -it busybox
  whoami
  ps
  ls /bin
  hostname
  date
  exit
> docker build -t hostname .
> docker run hostname
> docker run hostname date
> docker build -t entryhostname .
> docker run entryhostname
> docker run entryhostname date
> docker inspect entryhostname
> docker run --help
> docker run --entrypoint date entryhostname
```

- dockerfiles\entrypoint\entrypoint-1

```
FROM busybox:latest

ENTRYPOINT ["hostname"]

```
- **ENTRYPOINT** complements **CMD**
- Use CMD by default

### Using ENTRYPOINT and CMD together
- [MySQL Docker Hub Image](https://hub.docker.com/_/mysql)
- [SIGINT, SIGTERM, and SIGKIL](https://eitca.org/cybersecurity/eitc-is-lsa-linux-system-administration/linux-processes/process-signals/examination-review-process-signals/explain-the-difference-between-sigint-sigterm-and-sigkill-signals-in-linux/)
- [PIDs in Linux and Unix](https://en.wikipedia.org/wiki/Process_identifier)
- [Docker Blog: Choosing between RUN, CMD, and ENTRYPOINT](https://www.docker.com/blog/docker-best-practices-choosing-between-run-cmd-and-entrypoint/)
- If both ENTRYPOINT and CMD are set, they combine into a single command on container start.
- For CLI tools, use ENTRYPOINT to set the base executable, while CMD should provide default arguments.
- CMD can be easily overridden at docker run without replacing the ENTRYPOINT.
- For pre-launch scripts, ENTRYPOINT should set the script, and CMD should set the final process.
- ENTRYPOINT shell scripts should use exec “$@” to pass execution (PID 1) to the CMD.
- example 1

```
FROM ubuntu:latest

RUN apt-get update && \
	apt-get install -y --no-install-recommends \
	curl \
	&& rm -rf /var/lib/apt/lists/*
	
ENTRYPOINT ["curl"]

CMD ["--help"]	
```
- example 2

```
ENTRYPOINT ["httping", "-Y"]

CMD ["--version"]
```
- example running a script
```
ENTRYPOINT ["./startup.sh"]
CMD ["python", "app.py"]
```
- another process will be started if we add `exec "$@"'`
 
 ### Shell vs Exec Form
 - [(Docs) Shell and Exec Form](https://docs.docker.com/reference/dockerfile/#shell-and-exec-form)
- [(Docs) How CMD and ENTRYPOINT](https://docs.docker.com/reference/dockerfile/#understand-how-cmd-and-entrypoint-interact)
- [(Docs) The SHELL statement](https://docs.docker.com/reference/dockerfile/#shell)
- [(Docs) ENTRYPOINT Examples](https://docs.docker.com/reference/dockerfile/#entrypoint)
- [Crazy ENTRYPOINT, CMD, and SHELL examples](https://dev.to/rimelek/constructing-commands-to-run-in-docker-containers-2g2i)
- The RUN, ENTRYPOINT, and CMD instructions can be specified in shell form or exec form.
- Shell form will inject `/bin/sh -c` at the beginning of the command.
- Overwrite the shell that Docker injects with the SHELL statement, e.g. `SHELL ["/bin/bash", "-c"]`
- Shell form is needed for variable substitution, chaining commands, piping output, I/O redirection.
- Exec form (JSON syntax) runs the command without a shell.
- Exec form ensures ENTRYPOINT/CMD binary is PID 1 and receives signals.
- Exec form still passes ENVs from Dockerfile to processes started with ENTRYPOINT, CMD, and RUN.
- Don’t mix forms between ENTRYPOINT and CMD, or [weird things happen](https://docs.docker.com/reference/dockerfile/#understand-how-cmd-and-entrypoint-interact).
- General advice for which form to use:
    - RUN: Use Shell by default.
    - ENTRYPOINT: Always Exec, or CMD can’t be used.
    - CMD: Use Exec by default, but sometimes Shell Form is needed for shell features.
    - ENTRYPOINT + CMD: Always use Exec to avoid [weird edge cases](https://docs.docker.com/reference/dockerfile/#understand-how-cmd-and-entrypoint-interact).
    
- Exec form:
```
ENTRYPOINT["startup.sh"]
CMD ["python", "server.py"]
```
- Shell form:
```
SHELL["/bin/sh", "-c"]
```


```
# use 
SHELL["/bin/sh", "-c"]
# or
SHELL["powershell", "-command"]
```
  
### Assignment 1: Create CLI Utilities
#### Cmatrix
- Dockerfile
```
FROM alpine:latest

RUN apk add --no-cache cmatrix

ENTRYPOINT ["cmatrix"]

CMD ["-abs","-C", "red"]
```
- Running
```
docker run -it cmatrix

# or

docker run -it --entrypoint sh cmatrix
```
#### ApacheBench
- Dockerfile
```
FROM ubuntu:latest

RUN apt-get update && \
	apt-get install -y apache2-utils && \ 
	rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["ab"]

CMD ["-n", "10", "-c", "2", "https://www.bretfisher.com/"]
```
### Assignment 2: Startup Scripts
```
FROM python:slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

VOLUME "/app/data"

ENTRYPOINT ["./docker-entrypoint.sh"]

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Docker Compose: The Multi-Container Tool
- [Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Compose Releases](https://github.com/docker/compose/releases)
- [YAML](https://yaml.org/)
- Configures relationships between containers
- Saves our docker container run settings in easy-to-read file
- Creates one-liner developer environment startups
- Comprised of 2 things:
1. YAML-formatted file that describes our solution options for:
  - containers
  - networks
  - volumes
2. A CLI tool docker-compose used for local dev/test
automation with those YAML files

#### docker-compose.yml
- Compose YAML format has it's own versions: 1, 2, 2.1, 3, 3.1
- YAML file can be used with docker-compose command for
local docker automation or..
- With docker directly in production with Swarm (as of v1.13)
- docker-compose --help
- docker-compose.yml is default filename, but any can be
used with docker-compose -f

#### docker-compose CLI
- CLI tool comes with Docker for Windows/Mac, but separate
download for Linux
- Not a production-grade tool but ideal for local development and test
- Two most common commands are
  - docker-compose up # setup volumes/networks and start all containers
  - docker-compose down # stop all containers and remove cont/vol/net
- If all your projects had a Dockerfile and docker-compose.yml
then "new developer onboarding" would be:
  - git clone github.com/some/software
  - docker-compose up

#### Template
```yaml
# version isn't needed as of 2020 for docker compose CLI. 
# All 2.x and 3.x features supported
# Docker Swarm still needs a 3.x version
# version: '3.9'

services:  # containers. same as docker run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
  servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```
- Example 1
```
# version isn't needed as of 2020 for docker compose CLI. 
# All 2.x and 3.x features supported
# version: '2'

# same as 
# docker run --name jekyll -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve

services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site
    ports:
      - '80:4000'
```
- Example 2

```
# version isn't needed as of 2020 for docker compose CLI. 
# All 2.x and 3.x features supported
# version: '2'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: example
      WORDPRESS_DB_PASSWORD: examplePW
    volumes:
      - ./wordpress-data:/var/www/html

  mysql:
    # we use mariadb here for arm support
    # mariadb is a fork of MySQL that's often faster and better multi-platform
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: examplerootPW
      MYSQL_DATABASE: wordpress
      MYSQL_USER: example
      MYSQL_PASSWORD: examplePW
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:

```
- Exmaple 3
```
version: '3.9'
# NOTE: This example only works on x86_64 (amd64)
# Percona doesn't yet publish arm64 (Apple Silicon M1) or arm/v7 (Raspberry Pi 32-bit) images

services:
  ghost:
    image: ghost
    ports:
      - "80:2368"
    environment:
      - URL=http://localhost
      - NODE_ENV=production
      - MYSQL_HOST=mysql-primary
      - MYSQL_PASSWORD=mypass
      - MYSQL_DATABASE=ghost
    volumes:
      - ./config.js:/var/lib/ghost/config.js
    depends_on:
      - mysql-primary
      - mysql-secondary
  proxysql:
    # image only works on x86_64 (amd64)
    image: percona/proxysql
    environment: 
      - CLUSTER_NAME=mycluster
      - CLUSTER_JOIN=mysql-primary,mysql-secondary
      - MYSQL_ROOT_PASSWORD=mypass
   
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
  mysql-primary:
    # image only works on x86_64 (amd64)
    image: percona/percona-xtradb-cluster:5.7
    environment: 
      - CLUSTER_NAME=mycluster
      - MYSQL_ROOT_PASSWORD=mypass
      - MYSQL_DATABASE=ghost
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
  mysql-secondary:
    # image only works on x86_64 (amd64)
    image: percona/percona-xtradb-cluster:5.7
    environment: 
      - CLUSTER_NAME=mycluster
      - MYSQL_ROOT_PASSWORD=mypass
   
      - CLUSTER_JOIN=mysql-primary
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
    depends_on:
      - mysql-primary

```
