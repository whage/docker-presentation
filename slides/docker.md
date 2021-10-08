# Docker

---

Why we need containers

- microservices era - complex application stacks
- common dependencies, different versions on the same system
- need for a well-defined, repeatable runtime environment
- unit of application packaging
- greatly simplifies application deployment
- [history of containerization](https://blog.aquasec.com/a-brief-history-of-containers-from-1970s-chroot-to-docker-2016)

---

Background - Linux concepts

- a process is an executing instance of a program
- namespaces: a Linux kernel feature - isolation for processes
	- namespace types: mount, network, uid and a few others
	- at startup: a single namespace of each type, used by all processes
- control groups (cgroups): another Linux feature
	- setting resource limits (mainly CPU, RAM)

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
	- cpu, memory limits

---

What is Docker?

- containers != Docker
- Docker is a "container runtime": it manages containers for you
- other container runtimes: rkt, cri-o, podman, others...
- services provided by Docker:
	- running containers and managing their state
	- building images, managing remote repositories
	- managing persistence for containers
	- managing networking for containers

---

Docker architecture - Overview

![Docker architecture](images/docker-architecture.svg)

Source: [Docker overview](https://docs.docker.com/get-started/overview/)

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

![Overlay filesystem](images/julia-evans-overlay.jpeg) <!-- .element height="70%" width="70%" -->

Credit: [Julia Evans](https://jvns.ca/blog/2019/11/18/how-containers-work--overlayfs/)

---

Docker image layers

![Mount types](images/layers.png)

---

What it looks like

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

```Dockerfile
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

- OCI runtime spec & OCI image spec
- initiative for container runtime interchangeability
- a "Docker image" is really an "OCI image" today

```
docker run example.com/org/app:v1.0.0
rkt run example.com/org/app,version=v1.0.0
```

---

Container registries

- repository for container images
- like remote git repos but for images
- REST API
- private registries for private images with authentication

# TODO
- dockerhub with link, show the UI
- istos.azurecr.io, show UI

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

```sh
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

- running as root / rootless
- privileged
- image scanning

# TODO
- elaborate on rootless
- explain importance of image scanning
	- We have Jfrom X-ray
	- needs to be integrated into build pipelines

https://wiki.app.dmgmori.com/pages/viewpage.action?spaceKey=IPSTC&title=Docker+container+non-root+context

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
	- Docker cli
	- docker-compose

---

Useful commands

- `docker exec -it CONTAINER COMMAND`
- `docker logs -f CONTAINER`
- `docker cp CONTAINER:SRC_PATH DEST_PATH`

---

Useful links

- ["The container ecosystem"](https://accenture.github.io/blog/2021/03/25/docker-and-the-container-ecosystem.html)
- ["Docker behind the scenes"](https://accenture.github.io/blog/2021/03/10/docker-behind-the-scenes.html)
- ["Docker internals overview"](https://accenture.github.io/blog/2021/03/18/docker-components-and-oci.html)
- ["Namespaces in Go series"](https://medium.com/@teddyking/namespaces-in-go-basics-e3f0fc1ff69a)
- ["Deep dive into namespaces series"](http://ifeanyi.co/posts/linux-namespaces-part-1/)
