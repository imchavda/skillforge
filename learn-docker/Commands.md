# Docker Commands Cheat Sheet

## Key Concepts
1. Every Docker image has a unique image ID.
2. Every container has a unique container ID.
3. Container ID represents the container hostname.

---

## Image Management

- **Search for images on Docker Hub:**
  ```
  docker search <image-name>
  ```

- **List local images:**
  ```
  docker images
  ```

- **Pull/download an image:**
  ```
  docker pull <image-name>:<tag>
  ```

- **Save an image to a tar file:**
  ```
  docker save -o /path/to/image.tar <image-name>:<tag>
  ```

- **Load an image from a tar file:**
  ```
  docker load -i /path/to/image.tar
  ```

- **View image history:**
  ```
  docker history <image-id or image-name>
  ```

---

## Container Lifecycle

- **Run a new container (interactive):**
  ```
  docker container run -it --name <container-name> <image-name>
  ```
  - `-i`: Interactive
  - `-t`: Allocate a pseudo-TTY
  - `--name`: Assign a name to the container

- **Run a container in detached mode:**
  ```
  docker run -d --name <container-name> <image-name>:<tag>
  ```
  - `-d`: Detached (runs in background)

- **Run with port mapping:**
  ```
  docker run -d --name <container-name> -p <host-port>:<container-port> <image-name>:<tag>
  ```

- **Run with volume mount:**
  ```
  docker run -dit -P --name <container-name> -v /host/path:/container/path <image-name>:<tag>
  ```
  - `-v`: Mount a volume

---

## Container Management

- **List running containers:**
  ```
  docker ps
  ```

- **List all containers (including stopped):**
  ```
  docker container ls -a
  ```

- **List all containers with size:**
  ```
  docker container ls -a -s
  ```

- **Start/stop/remove a container:**
  ```
  docker start <container-id or name>
  docker stop <container-id or name>
  docker rm <container-id or name>
  ```

- **Access a running container shell:**
  ```
  docker exec -it <container-id or name> /bin/sh
  docker exec -it <container-id or name> /bin/bash
  ```

- **Inspect a container:**
  ```
  docker container inspect <container-id or name>
  ```

- **View container logs:**
  ```
  docker container logs <container-id or name>
  ```

- **Show container port mappings:**
  ```
  docker port <container-id or name>
  ```

- **Show top processes in a container:**
  ```
  docker top <container-id or name>
  ```

---

## Container States

1. Created
2. Running
3. Paused (pause/unpause)
4. Stopped
5. Killed

- **Pause/unpause a container:**
  ```
  docker pause <container-id>
  docker unpause <container-id>
  ```

- **Kill a running container:**
  ```
  docker kill <container-id>
  ```

---

## Docker Version Info

- **Check Docker version:**
  ```
  docker version
  ```

- **Check Docker Compose version:**
  ```
  docker-compose --version
  ```

- **Check Docker Machine version:**
  ```
  docker-machine --version
  ```

---

## Notes

- If an image is not found locally, Docker will pull it from Docker Hub.
- Use `-P` to publish all exposed ports to random host ports.
- Use `-p` to map specific host ports to container ports.
- Container IDs and names can be used interchangeably in most commands.
- Use `docker history <image>` to see all commands run to build an image.

---

