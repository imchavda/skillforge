# Docker Networking and DNS

---

## Docker DNS and Service Discovery

- Use `--link` or `--net-alias` to enable DNS-based service discovery between containers.
- **Best practice:** Containers should communicate using DNS names, not IP addresses.

### Example: Running Containers with DNS Alias

```sh
docker run -d --net my_app_net --net-alias abc.com elasticsearch:2
```

### Check DNS Resolution and Connectivity

```sh
docker run --rm --net my_app_net alpine nslookup abc.com
docker run --rm --net my_app_net alpine ping abc.com
```

---

## Docker Networks Overview

- Each container is connected to a private virtual network (default: `bridge`).
- Each virtual network is routed through a NAT firewall on the host IP.
- All containers on the same virtual network can communicate without explicit port mapping (`-p`).
- **Best practice:** Create a dedicated virtual network for each application stack.
    - Example:  
      - `my_web_app` for MySQL and PHP/Apache containers  
      - `my_api` for MongoDB and Node.js containers

---

## Managing Docker Networks

### Create a New Virtual Network

```sh
docker network create my_app_net
```

### Run a Container on a Specific Network

```sh
docker run -d -p 80:80 --name nginx1 --network my_app_net nginx
```

### Attach/Detach a Container to/from a Network

```sh
docker network connect my_app_net <container>
docker network disconnect my_app_net <container>
```
- A container can be attached to multiple networks and will have multiple IPs.

### Use Host Networking (Advanced)

- Skip Docker's virtual networks and use the host's network stack:
    ```sh
    docker run --net=host ...
    ```
- **Note:** This improves performance but reduces container isolation and security.

---

## Docker Network Drivers

- Use different network drivers to enable various networking features.
    - Common drivers: `bridge`, `host`, `overlay`, `macvlan`, etc.

---

## Useful Docker Network Commands

- **List all networks:**
    ```sh
    docker network ls
    ```
- **Inspect a network:**
    ```sh
    docker network inspect <network-name>
    ```
- **Create a network with a specific driver:**
    ```sh
    docker network create --driver <driver-name> <network-name>
    ```
- **Attach a network to a running container:**
    ```sh
    docker network connect <network-name> <container>
    ```
- **Detach a network from a running container:**
    ```sh
    docker network disconnect <network-name> <container>
    ```

---

## Example: Creating and Using a Custom Network

1. **Create a new bridge network:**
    ```sh
    docker network create --driver bridge new_nw
    ```

2. **Run a container on the new network:**
    ```sh
    docker run -it --network=new_nw ubuntu:latest /bin/bash
    ```

3. **Inspect the network:**
    ```sh
    docker network inspect new_nw
    ```

---

## Summary

- Use Docker networks to isolate and manage container communication.
- Prefer DNS names over IPs for service discovery.
- Leverage network drivers and aliases for flexible networking setups.