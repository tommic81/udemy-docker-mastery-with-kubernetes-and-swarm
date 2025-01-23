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