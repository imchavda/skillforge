# Docker: Container Linking Example

This guide demonstrates how to link two Docker containers so that one can communicate with the other by an alias.

---

## 1. Pull the Jenkins Image

```sh
docker pull jenkins
```

## 2. Run the Jenkins Container

```sh
docker run --name jenkinsa -d jenkins
```
- `--name jenkinsa` : Names the container "jenkinsa".
- `-d` : Runs the container in detached mode.

---

## 3. Launch a Destination Container and Link to Jenkins

```sh
docker run --name reca --link jenkinsa:alias-src -it ubuntu:latest /bin/bash
```
- `--name reca` : Names the new container "reca".
- `--link jenkinsa:alias-src` : Links "jenkinsa" to "reca" with the alias "alias-src".
- `-it` : Interactive terminal.
- `ubuntu:latest /bin/bash` : Starts a bash shell in the Ubuntu container.

> Inside the "reca" container, you can now access "jenkinsa" using the hostname `alias-src`.

---

## 4. List Running Containers

```sh
docker ps
```

---

**Note:**  
The `--link` flag is a legacy feature and may be removed in future Docker versions. For modern container communication, consider using Docker networks.
