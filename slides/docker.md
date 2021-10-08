# Docker

---

What we will talk about

- history of containers, how we got here
- backgroudn on containers (Linux internals)
- Docker images, Dockerfile, persistent storage
- common use cases - docker-compose, Docker desktop
- security
- useful commands

---

History of Docker & containers

# TODO

---

Background - Linux concepts

- a process is an executing instance of a program
- namespaces: a Linux kernel concept - isolation for processes
	- fixed number of namespace types: mount, network, uid and others
	- at startup: a single namespace of each type, used by all processes
- control groups: for setting resource limits (mainly CPU, RAM)

---

What is a Docker container?

- a regular process that is namespaced separately from the others
- containers (they are simple processes!) share the same kernel

![Shared kernel](images/shared-kernel.png) <!-- .element height="40%" width="40%" -->

---

Isolation

- Docker creates a set of namespaces and control groups for the container
- isolation provided by namespaces
	- mount namespace: isolated file system
	- network namespace: isolated networking
- resource limits provided by control groups (cgroups)
	- another Linux kernel feature
	- cpu, memory limits

---

What is Docker?

- containers != Docker
- Docker is a "container runtime": it manages containers for you
- other container runtimes: rkt, cri-o, podman, others...
- services provided by Docker:
	- running containers and managing their state
	- managing images - image build (Dockerfile), remote repositories
	- managing persistence for containers
	- networking for containers

---

Docker architecture - Overview

![Docker architecture](images/docker-architecture.svg)

Source: [Docker overview](docker-overview)

---

Docker images

- a file system bundle, basis for a running container
- defines the whole file system hierarchy for the containers
	- fixed executable versions
	- fixed library versions, dependencies
- basis for a predictable, stable environment
- image naming `<repository>:<tag>`, implicit push target

---

Union filesystems

- Docker images use this format
- the containerized app doesn't know, doesn't care
- like stacking layers of film on each other
- efficient storage
	- the bottom layers can be shared
	- only the deltas are recorder in new layers
- only the top layer is writable, only for the duration of the container

---

Union filesystem in Docker

![Overlay filesystem](images/julia-evans-overlay.jpeg) <!-- .element height="60%" width="60%" -->

Credit: [Julia Evans](https://jvns.ca/blog/2019/11/18/how-containers-work--overlayfs/)

---

Layers

![Mount types](images/layers.png)

---

Overlay

![Mount types](images/overlay.png)

---

Dockerfile

- the Dockerfile a text file with instructions on how to build a Docker image
- `docker build` - needs a Dockerfile and a context
- custom, declarative syntax
- `FROM` directive: build on top of an existing image

---

Dockerfile

![Mount types](images/dockerfile.png)

---

Multi-stage builds

```
FROM node:12.13.0-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install

# Waste lots of disk space with other things
...

FROM nginx
EXPOSE 3000
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/build /usr/share/nginx/html
```

---

OCI - Open Container Initiative

# TODO

- OCI runtime spec
- OCI image spec

---

Container registries

- repository for container images
- like remote git repos but for images
- REST API
- private registries for private images with authentication

---

Persistent storage

- problem: the writable layer is lost when containers are killed
- solution for persistence: separate storage mechanisms: volumes
- independent of container lifecycle
- other options: bind mount, tmpfs

![Mount types](images/mount-types.png) <!-- .element height="35%" width="35%" -->

---

Volumes

- managed by Docker
- data is stored on the host fs, mapped into the container
- can be shared by multiple containers

![Mount types](images/var-lib-docker-volumes.jpg) <!-- .element height="35%" width="35%" -->

---

Example run

```
docker run \
  --rm \
  --name nginx \
  -v /home/vagrant/work/tmp/docker-presentation/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /home/vagrant/work/tmp/docker-presentation:/var/www:ro \
  -p 9999:80 \
  -d \
  nginx
```

---

Container security

# TODO

- rootless
- privileged

---

docker-compose

- for multi-container applications
- everything configured in a single yaml file
	- containers
	- networks
	- volumes
- runs Docker commands in the background
	- `docker-compose up`
	- `docker-compose down`

---

Docker desktop - Windows & Mac

- containers are based on Linux kernel features
- requires either WSL2 or Hyper-V
- Docker desktop includes
	- Docker engine
	- Docker clie
	- docker-compose

---

Useful commands

- `docker exec -it CONTAINER COMMAND`
- `docker logs -f CONTAINER`
- `docker cp CONTAINER:SRC_PATH DEST_PATH`

---

Useful links

- ["The container ecosystem"](container-ecosystem)
- ["Docker behind the scenes"](docker-behind)
- ["Docker internals overview"](docker-runc)

[docker-overview]: https://docs.docker.com/get-started/overview/
[docker-behind]: https://accenture.github.io/blog/2021/03/10/docker-behind-the-scenes.html
[docker-runc]: https://accenture.github.io/blog/2021/03/18/docker-components-and-oci.html
[container-ecosystem]: https://accenture.github.io/blog/2021/03/25/docker-and-the-container-ecosystem.html 
