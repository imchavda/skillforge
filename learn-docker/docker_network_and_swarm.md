# Docker Networking and Swarm Guide

---

## Docker Network

### List Docker Bridge Interfaces

```sh
brctl show
```

### Establish Connectivity Between Two Containers

1. **Start a Redis container:**
    ```sh
    docker pull redis
    docker run -d --name redis1 redis
    ```

2. **Start a BusyBox container and link it to Redis:**
    ```sh
    docker pull busybox
    docker run -it --link redis1:redis --name redisclient1 busybox
    ```

---

## Docker Swarm

### Prerequisites

1. Each machine must be able to communicate with the Swarm manager.
2. Docker must be installed and running on all nodes.

---

### Swarm Manager Setup

- **Initialize Swarm Manager:**
    ```sh
    docker swarm init --advertise-addr <MANAGER-IP>
    ```
    - Replace `<MANAGER-IP>` with your manager node's IP address.
    - This command generates a token used to join worker nodes.

- **Verify Swarm Manager Status:**
    ```sh
    docker info
    ```
    - Check the "Swarm" section for manager status.

- **List Swarm Nodes:**
    ```sh
    docker node ls
    ```

- **Remove a Node:**
    ```sh
    docker node rm <node_id>
    ```

- **Swarm Cluster Port:**  
    Default port is `2377`.

---

### Swarm Node Operations

- **Get Join Command for Worker Nodes (run on manager):**
    ```sh
    docker swarm join-token worker
    ```
    - Use the provided command on worker nodes to join the swarm.

- **Leave Swarm (run on node):**
    ```sh
    docker swarm leave
    ```

---

### Docker Services in Swarm

- **List Services:**
    ```sh
    docker service ls
    ```

- **Create a Service:**
    ```sh
    docker service create --replicas 1 --name helloworld alpine ping google.com
    ```
    - `--name helloworld`: Names the service.
    - `--replicas 1`: Runs one instance.
    - `alpine ping google.com`: Runs Alpine Linux and pings google.com.

- **Inspect a Service:**
    ```sh
    docker service inspect --pretty helloworld
    docker service ps helloworld
    ```

- **List Containers:**
    ```sh
    docker ps
    ```

- **Scale a Service:**
    ```sh
    docker service scale helloworld=5
    ```

- **Check Service Tasks:**
    ```sh
    docker service ps helloworld
    ```

- **Check Running Containers on Worker Nodes:**
    ```sh
    docker ps
    ```

---

### Service and Node Maintenance

- **Remove a Service:**
    ```sh
    docker service rm <servicename>
    ```

- **Put Worker Node in Maintenance Mode:**
    ```sh
    docker node update --availability drain <worker-node-name>
    ```

---