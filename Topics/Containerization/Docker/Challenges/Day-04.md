# Day 4: Docker Networking - Fundamentals & Host Communication

## **Concepts:**
  * **Container Networking:** How Docker containers communicate with each other and with the outside world.
  * **Isolated Network Stack:** Each Docker container by default gets its own isolated network stack (IP address, routing table, DNS).
  * **Bridge Network (Default):** The most common network type. Containers on the same bridge network can communicate by IP address. Docker automatically creates a default `bridge` network.
  * **Host Network:** Container shares the host's network stack. No network isolation between the container and the host.
  * **None Network:** Disables all networking for the container.
  * **Port Mapping (`-p` or `--publish`):** The mechanism to map a port from the container's network stack to a port on the Docker host's network stack, allowing external access.

## **Examples:**

1.  **Inspect Docker's default networks:**
```bash
docker network ls
docker network inspect bridge
```
(Note the `Subnet` and `Gateway` of the `bridge` network.)

2.  **Run a simple web server on the default bridge network (without port mapping):**
```bash
docker run -d --name nginx_internal nginx:latest
docker inspect nginx_internal | grep "IPAddress"
```
(You won't be able to access this Nginx from your host browser because no port is mapped.)

3.  **Run Nginx with port mapping (default bridge network):**
```bash
docker run -d -p 8080:80 --name nginx_web nginx:latest
```
(Access `http://localhost:8080` in your browser. This maps host port 8080 to container port 80.)

4.  **Explore host networking:**
```bash
docker run -it --network host alpine:latest ip addr show eth0
```
(The container's IP address will be the same as your host's primary IP address. Be careful, as this removes network isolation.)

5.  **Explore none networking:**
```bash
docker run -it --network none alpine:latest ip addr show
```
(You'll see no network interfaces configured other than the loopback.)

## **Challenge 4: Port Mapping & Container Reachability**

* Run your Flask application from Day 2 in a container, but this time, map its internal port (e.g., 5000) to two different host ports (e.g., 8081 and 8082). Can you access it from both host ports?
* Run two separate Nginx containers, both exposing port 80 internally, but map them to different host ports (e.g., 8083 and 8084). Verify you can reach both.
* Try to run a container using `--network host` that starts a web server on port 80. What happens if your host machine already has a web server running on port 80?

