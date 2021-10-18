# Docker

---

Why we need containers

- microservices era - complex application stacks
- common dependencies, different versions on the same system
- need for a well-defined, repeatable runtime environment
- need for a unit of application packaging
- containers greatly simplify application deployment by encapsulation
- [history of containerization](https://blog.aquasec.com/a-brief-history-of-containers-from-1970s-chroot-to-docker-2016)

---

Background on containers

- Docker containers are based on Linux concepts
- processes: an executing instance of a program
- namespaces: a Linux kernel feature - isolation for processes
	- namespace types: mount, network, uid and a few others
	- at startup: a single namespace of each type, used by all processes
- control groups (cgroups): another Linux feature
	- setting resource limits (mainly CPU, RAM)

---

What is a Docker container?

- a regular process that is namespaced separately from the others
- "lightweight": no different from any other process in the system! just processes!

![Shared kernel](images/shared-kernel.png) <!-- .element height="40%" width="40%" -->

---

Containers provide isolation and resource limits

- Docker creates a set of namespaces and control groups for the container
- isolation provided by namespaces, for example:
	- mount namespace: isolated file system
	- network namespace: isolated networking
- resource limits provided by control groups (cgroups)
	- cpu, memory limits

---

What is Docker?

- containers != Docker
- Docker is a "container runtime": it manages containers for you
- other container runtimes: rkt, cri-o, podman, others
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

---

Union filesystems

- Docker images use this format, but it is independent of containers
- main advantage: efficient storage
	- images can share sets of layers
	- only deltas are recorder in new layers
- only the top layer is writable during run-time
	- containers don't modify their images
- the containerized app inside doesn't know, doesn't care!

---

Union filesystem in Docker

![Overlay filesystem](images/julia-evans-overlay.jpeg) <!-- .element height="70%" width="70%" -->

Credit: [Julia Evans](https://jvns.ca/blog/2019/11/18/how-containers-work--overlayfs/)

---

What it looks like

![Mount types](images/overlay.png)

---

Shared layers

![Mount types](images/layers.png)

---

Dockerfile

- a text file with instructions on how to build a Docker image
- `docker build`
	- needs a Dockerfile and a context
- custom, declarative syntax
- `FROM` directive: "inheritance"
	- builds on top of an existing image

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

- initiative for container runtime interchangeability
- 2 specifications:
	- runtime spec
	- image spec
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
	- `docker login`
- image names imply the target repository: `<repository>:<tag>`

---

Container registries that we use

- the official Docker image registry: [dockerhub](https://hub.docker.com/)
- our internal Azure container registry (ACR): [istos.azurecr.io](https://portal.azure.com/#@dmgmori.onmicrosoft.com/resource/subscriptions/7feec180-3aa3-44f9-a1e0-790204d56882/resourceGroups/ne-dz-rg0001/providers/Microsoft.ContainerRegistry/registries/istos/overview)

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
- data is persisted on the host fs, mapped into the container
- can be shared by multiple containers

![Mount types](images/var-lib-docker-volumes.jpg) <!-- .element height="35%" width="35%" -->

---

Docker container networking

- containers run in a different network namespace
	- isolated from the host
- Docker creates the docker0 bridge interface on the host
- connect to other network namespace via virtual ethernet (veth) pairs
- container ports -> host ports need explicit mapping 

---

Docker container networking

![Container networking](images/container-networking.png) <!-- .element height="60%" width="60%" -->

Credit: [DZone](https://dzone.com/articles/step-by-step-guide-establishing-container-networki)

---

Example: running a container

```sh
docker run \
  --rm \
  --name nginx \
  -v /docker-presentation/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /docker-presentation:/var/www:ro \
  -p 9999:80 \
  -d \
  nginx
```

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

Container security

- [not secure by default](https://blog.gitguardian.com/how-to-improve-your-docker-containers-security-cheat-sheet/)!
- the Docker daemon runs as root
	- it can do anything when starting containers
- [rootless alternatives](https://developers.redhat.com/blog/2020/09/25/rootless-containers-with-podman-the-basics#): podman, kaniko, img, buildah
- the default user [inside Docker containers is root](https://amazicworld.com/get-the-evil-out-dont-run-containers-as-root/)
	- problems with sensitive mounts
	- [privilege escalations](https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation)

---

Security best practices

- use trusted base images -> [Docker Official Images](https://hub.docker.com/search?q=&type=image&image_filter=official)
- [run containers as a non-root user](https://www.redhat.com/en/blog/understanding-root-inside-and-outside-container)
	- needs a carefully built image
- don't store any sensitive information in images
- scan images!
	- discovering vulnerable dependencies
	- we have Jfrom X-ray but needs to be integrated into pipelines

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
- `docker inspect CONTAINER`

---

Useful links

- ["The container ecosystem"](https://accenture.github.io/blog/2021/03/25/docker-and-the-container-ecosystem.html)
- ["Docker behind the scenes"](https://accenture.github.io/blog/2021/03/10/docker-behind-the-scenes.html)
- ["Docker internals overview"](https://accenture.github.io/blog/2021/03/18/docker-components-and-oci.html)
- ["Namespaces in Go series"](https://medium.com/@teddyking/namespaces-in-go-basics-e3f0fc1ff69a)
- ["Deep dive into namespaces series"](http://ifeanyi.co/posts/linux-namespaces-part-1/)
