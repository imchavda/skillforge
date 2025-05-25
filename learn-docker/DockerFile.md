# Dockerfile Example and Instructions

## Sample Dockerfile

```dockerfile
# Use Ubuntu 19.10 as the base image
FROM ubuntu:19.10

# Update package list and install Apache2
RUN apt-get update
RUN apt-get install -y apache2 apache2-utils
RUN apt-get clean

# Expose port 80
EXPOSE 80

# Start Apache in the foreground
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

> Save this file as `Dockerfile`.

---

## Building the Docker Image

```sh
docker build -t myimage:0.1 .
```
- `-t` : Assigns a tag to the image (`myimage:0.1`).
- `.` : Specifies the build context (current directory).

---

## Running the Container

```sh
docker run -d -p 80:80 myimage:0.1
```
- `-d` : Runs the container in detached mode (in the background).
- `-p 80:80` : Maps port 80 of the host to port 80 of the container.

---

## Checking Exposed Ports

To inspect which ports are exposed by an image:
```sh
docker inspect <image-name>
```
Example:
```sh
docker inspect jenkins
```

---

## Dockerfile Instruction Commands

### 1. CMD

- **Purpose:** Executes a command at runtime when the container starts.
- **Syntax:**  
  ```dockerfile
  CMD ["executable", "param1", "param2"]
  ```
- **Example:**  
  ```dockerfile
  CMD ["apache2ctl", "-D", "FOREGROUND"]
  ```

### 2. ENV

- **Purpose:** Sets environment variables in the container.
- **Syntax:**  
  ```dockerfile
  ENV key value
  ```
- **Example:**  
  ```dockerfile
  FROM ubuntu
  MAINTAINER demousr@gmail.com
  ENV var1=Tutorial var2=point
  ```

### 3. WORKDIR

- **Purpose:** Sets the working directory for any RUN, CMD, ENTRYPOINT, COPY, and ADD instructions that follow.
- **Syntax:**  
  ```dockerfile
  WORKDIR /path/to/directory
  ```
- **Example:**  
  ```dockerfile
  FROM ubuntu
  MAINTAINER demousr@gmail.com
  WORKDIR /newtemp
  CMD pwd
  ```

---

**Notes:**
- If the directory specified in `WORKDIR` does not exist, it will be created.
- You can use `docker inspect <container-id or image-name>` to view detailed information about images and containers.
