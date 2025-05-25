# Docker Storage Overview

---

## 1. Storage Drivers

| Technology   | Storage Driver   |
|--------------|-----------------|
| OverlayFS    | overlay, overlay2|
| AUFS         | aufs            |
| Btrfs        | btrfs           |
| Device Mapper| devicemapper    |
| VFS          | vfs             |
| ZFS          | zfs             |

### When to Use Each Storage Driver

- **AUFS**
  - Stable and production-ready.
  - Good memory usage and smooth Docker experience.
  - High write activity; suitable for Platform-as-a-Service (PaaS) systems.

- **Device Mapper**
  - Stable and aligns with mainline Linux kernel.
  - Good for testing applications in labs.

- **Btrfs**
  - Kernel-integrated.
  - Handles high write activity.
  - Useful for managing multiple build pools.

- **Overlay/Overlay2**
  - Stable and kernel-integrated.
  - Good memory usage.
  - Suitable for testing applications in labs.

- **ZFS**
  - Stable and good for lab testing.
  - Suitable for PaaS systems.

---

## 2. Data Volumes

Data volumes are special directories outside the container’s Union File System (UFS) that persist data and can be shared across containers.

**Key Features:**
1. Initialized when the container is created.
2. Can be shared and reused among containers.
3. Changes to the volume are made directly.
4. Persist even after the container is deleted.
5. Require manual deletion.

**Inspecting Volumes:**
- You can see volume paths in the Dockerfile or by running:
  ```
  docker image inspect <image-name>
  ```

**Named Volumes Example:**
```sh
docker run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=True -v mysql-db:/var/lib/mysql mysql
```

---

## 3. Bind Mounts

Bind mounts map a host file or directory to a container path.

- Skips UFS; host files overwrite container files.
- Cannot be defined in a Dockerfile; must be specified at container run.

**Example:**
```sh
docker run -d --name nginx -v /home/devops/nginx:/usr/share/nginx/html -p 80:80 nginx
docker run -d -v /home/demo:/var/jenkins_home -p 8080:8080 -p 50000:50000 jenkins
```
After launching, files in `/home/demo` on the host will reflect the container’s `/var/jenkins_home` directory.

---

## 4. Docker Volumes

Docker volumes are managed by Docker and can be created, listed, and used independently of containers.

**Create a Volume:**
```sh
docker volume create --name volumename --opt options
```
- `--name` : Name of the volume.
- `--opt`  : Additional options (e.g., size).

**Example:**
```sh
docker volume create --name=demo --opt o=size=100m
```

**List Volumes:**
```sh
docker volume ls
```

---

## 5. Using Custom Storage Drivers

You can specify a storage driver for a container at runtime using the `--volume-driver` option.

**Example:**
```sh
docker run -d --volume-driver=flocker -v /home/demo:/var/jenkins_home -p 8080:8080 -p 50000:50000 jenkins
```
- `--volume-driver` : Specifies the storage driver.
- `-v` : Maps the host directory to the container.

---

## 6. Database Migration Example

**Create a Docker volume:**
```sh
docker volume create psqldb
```

**Run a PostgreSQL container using the volume:**
```sh
docker run -d -e POSTGRES_PASSWORD=p@ssw0rd -p 5432:5432 --name psql9.6 -v psqldb:/var/lib/postgresql/data postgres:9.6
```
or with a specific version:
```sh
docker run -d -e POSTGRES_PASSWORD=p@ssw0rd -p 5432:5432 --name psql9.6 -v psqldb:/var/lib/postgresql/data postgres:9.6.20
```

---

**Summary:**
- Use **volumes** for persistent, shareable data.
- Use **bind mounts** for direct host-to-container file mapping.
- Choose the appropriate **storage driver** for your environment and workload.
