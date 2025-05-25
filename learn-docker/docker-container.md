# Docker: Images, Containers, Networking, and Storage

---

## What is an Image?

- An **image** is built from libraries and source code that make up an application.
- It contains app binaries and all dependencies required to run the application.

---

## What is a Container?

- A **container** is a running instance of an image, executed as a process.
- Multiple containers can be created from a single Docker image.

---

## What Happens During `docker run`?

1. Docker checks for the image locally.
2. If not found, it pulls the image from a remote repository.
3. Downloads the latest version if needed.
4. Creates a new container from the image.
5. Assigns a virtual IP on Docker's private network.
6. Maps specified ports (e.g., host port 80 to container port 80).
7. Starts the container using the `CMD` defined in the Dockerfile.

---

## Standalone Containers vs. Services

- `docker run` creates a standalone container.
- `docker service create` (Swarm) creates service tasks (containers) across a cluster of nodes. Services act as templates for these tasks.

---

## Common Docker Commands

- **Run MySQL container:**
  ```sh
  docker run -d -p 3306:3306 -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
  ```
- **Show processes in a container:**
  ```sh
  docker top <container>
  ```
- **Show performance stats:**
  ```sh
  docker stats
  ```
- **Inspect container details:**
  ```sh
  docker inspect <container>
  ```
- **Get help for container run:**
  ```sh
  docker container run --help
  ```

---

## Accessing the Shell Inside a Container

- **Start a container with interactive shell:**
  ```sh
  docker run -it <image>
  ```
- **Execute a shell in a running container:**
  ```sh
  docker exec -it <container> /bin/bash
  docker exec -it <container> /bin/sh
  ```

---

## Docker Networking

- Each container is connected to a private virtual network (bridge) by default.
- All containers on the same bridge network can communicate without explicit port mapping.
- Best practice: create a dedicated virtual network for each application stack.

### Example: Creating and Using Networks

- **Create a new network:**
  ```sh
  docker network create my_app_net
  ```
- **Run a container on a specific network:**
  ```sh
  docker run -d -p 80:80 --name nginx1 --network my_app_net nginx
  ```
- **Attach/detach a container to/from a network:**
  ```sh
  docker network connect my_app_net <container>
  docker network disconnect my_app_net <container>
  ```

### Useful Network Commands

- List networks: `docker network ls`
- Inspect a network: `docker network inspect <network>`
- Create a network: `docker network create --driver <driver> <name>`
- Connect/disconnect a container: `docker network connect|disconnect <network> <container>`

---

## Docker DNS and Aliases

- Use `--link` or `--net-alias` to enable DNS-based service discovery.
- Containers should use DNS names, not IPs, for inter-communication.

**Example:**
```sh
docker run -d --net my_app_net --net-alias abc.com elasticsearch:2
docker run --rm --net my_app_net alpine nslookup abc.com
docker run --rm --net my_app_net alpine ping abc.com
```

---

## Docker Images

- Docker images use a union file system and are composed of layers, each with a unique SHA.
- Use `docker history <image>` to view image layers.

**Tagging and Pushing Images:**
```sh
docker image tag nginx:latest username/nginx:QA
docker login
docker image push username/nginx:QA
```

**Modify a tag:**
```sh
docker image tag username/nginx:QA username/nginx:testing
```

**Credentials file:**  
`cat /home/devops/.docker/config.json`

**Logout:**
```sh
docker logout
```

**Build a custom image:**
```sh
docker image build -t customnginx .
```

---

## Cleanup Commands

- Check Docker storage usage: `docker system df`
- Remove unused images: `docker image prune`
- Remove unused data (images, containers, networks): `docker system prune`

---

## Container Lifetime and Persistent Data

Containers are typically immutable and ephemeral. Use volumes or bind mounts for persistent data.

### 1. Data Volumes

- Special directories outside the container's UFS.
- Persist data even after container removal.
- Must be deleted manually.
- Can be named and shared between containers.

**Example:**
```sh
docker run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=True -v mysql-db:/var/lib/mysql mysql
```
- Inspect volume paths: `docker image inspect <image>`

### 2. Bind Mounts

- Map a host file or directory to a container path.
- Skips UFS; host files overwrite container files.
- Must be specified at container run, not in Dockerfile.

**Example:**
```sh
docker run -d --name nginx -v /home/devops/nginx:/usr/share/nginx/html -p 80:80 nginx
```
--- 