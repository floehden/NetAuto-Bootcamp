## Target Audience

This tutorial is designed for developers, operations professionals, and anyone interested in mastering Docker. It starts from the absolute basics and gradually moves to more advanced concepts, with a particular emphasis on how containers communicate. A basic understanding of the command line is helpful.

## What is Docker?

Docker is an open-source platform that automates the deployment, scaling, and management of applications using containerization. Containers package an application and all its dependencies (libraries, system tools, code, runtime) into a single, standardized unit. This ensures that the application runs consistently across different environments, from a developer's laptop to production servers.

**Why use Docker?**

  * **Consistency:** Eliminates "It works on my machine" issues.
  * **Isolation:** Applications run in isolated environments, preventing conflicts.
  * **Portability:** Easily move applications between different environments.
  * **Efficiency:** Containers are lightweight and start quickly, making efficient use of resources.
  * **Scalability:** Easier to scale applications by spinning up more containers.
  * **Version Control:** Dockerfiles and images are versionable, just like code.

-----

### Day 1: Docker Fundamentals - Images & Containers

**Concepts:**

  * **Images:** Read-only templates used to create containers. They contain the application code, libraries, dependencies, and configurations.
  * **Containers:** Runnable instances of an image. They are isolated, lightweight environments where your application runs.
  * **Docker Daemon:** The background service that manages Docker objects.
  * **Docker Client:** The command-line tool (or other clients) that interacts with the Docker Daemon.
  * **Docker Hub:** A public registry for Docker images.

**Examples:**

1.  **Install Docker:** Follow the official Docker documentation for your operating system (Docker Desktop for Windows/macOS, or Docker Engine for Linux).
2.  **Verify Installation:**
    ```bash
    docker --version
    docker run hello-world
    ```
    (The `hello-world` command pulls a tiny image and runs a container from it, printing a message.)
3.  **Pull an image:**
    ```bash
    docker pull ubuntu:latest
    ```
4.  **Run a container from an image (non-interactive):**
    ```bash
    docker run ubuntu:latest echo "Hello from Ubuntu container!"
    ```
5.  **Run an interactive container:**
    ```bash
    docker run -it --name my_ubuntu_shell ubuntu:latest /bin/bash
    ```
    (This runs an Ubuntu container in interactive (`-i`) and pseudo-TTY (`-t`) mode, names it `my_ubuntu_shell`, and opens a bash shell inside it. Type `exit` to leave the container.)
6.  **List running containers:**
    ```bash
    docker ps
    ```
7.  **List all containers (running and stopped):**
    ```bash
    docker ps -a
    ```
8.  **Stop a running container:**
    ```bash
    docker stop my_ubuntu_shell
    ```
9.  **Restart a stopped container:**
    ```bash
    docker start my_ubuntu_shell
    docker attach my_ubuntu_shell # Re-attach to the bash session
    ```
10. **Remove a container:**
    ```bash
    docker rm my_ubuntu_shell
    ```
11. **List images:**
    ```bash
    docker images
    ```
12. **Remove an image:**
    ```bash
    docker rmi ubuntu:latest
    ```

**Challenge 1: First Container Interaction**

  * Run a `debian:latest` container in interactive mode and try to install a package like `curl` inside it. What happens and why? (Hint: Think about what's included in a minimal Debian image.)
  * After exiting, try to start and re-attach to the same container.
  * Clean up by removing the container.

-----

### Day 2: Dockerfiles - Building Custom Images

**Concepts:**

  * **Dockerfile:** A text file containing instructions for building a Docker image. Each instruction creates a layer in the image.
  * **Build Context:** The directory containing the Dockerfile and any files it needs to copy into the image.
  * **Layers & Caching:** Docker images are composed of layers, where each instruction in a Dockerfile adds a new layer. This allows for efficient caching during builds.
  * **Common Dockerfile Instructions:** `FROM`, `WORKDIR`, `COPY`, `ADD`, `RUN`, `CMD`, `ENTRYPOINT`, `EXPOSE`.

**Examples:**

1.  **Create a simple Node.js application:**
    Create a directory named `my-node-app`. Inside, create `app.js`:
    ```javascript
    // app.js
    const http = require('http');
    const port = 3000;

    const server = http.createServer((req, res) => {
      res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
      res.end('Hello from Docker!\n');
    });

    server.listen(port, () => {
      console.log(`Server running at http://localhost:${port}/`);
    });
    ```
    And `package.json`:
    ```json
    {
      "name": "my-node-app",
      "version": "1.0.0",
      "description": "A simple Node.js app",
      "main": "app.js",
      "scripts": {
        "start": "node app.js"
      },
      "author": "",
      "license": "ISC"
    }
    ```
2.  **Create a `Dockerfile` in the same directory:**
    ```dockerfile
    # Use an official Node.js runtime as a parent image
    FROM node:18-alpine

    # Set the working directory in the container
    WORKDIR /usr/src/app

    # Copy package.json and package-lock.json to install dependencies
    # This is done first to leverage Docker's build cache for dependencies
    COPY package*.json ./

    # Install app dependencies
    RUN npm install

    # Copy the rest of the application code
    COPY . .

    # Expose the port the app runs on (documentation for humans, not functional)
    EXPOSE 3000

    # Define the command to run the application when the container starts
    CMD [ "npm", "start" ]
    ```
3.  **Build the Docker image:**
    ```bash
    docker build -t my-node-app:1.0 .
    ```
    (The `-t` flag tags the image with a name and optional version, and `.` specifies the build context.)
4.  **Run a container from your custom image:**
    ```bash
    docker run -p 8000:3000 --name my_running_app my-node-app:1.0
    ```
    (Access `http://localhost:8000` in your browser. Use `Ctrl+C` to stop the container.)

**Challenge 2: Containerize a Python Flask App**

  * Create a simple Python Flask application that returns "Hello from Flask in Docker\!" on port 5000.
  * Write a `Dockerfile` to containerize this Flask application.
  * Build the image and run a container, ensuring you can access the Flask app from your host browser.
  * Experiment with `CMD` vs `ENTRYPOINT`. What happens if you try to `docker run <your_image_name> echo "Hello"` when using `CMD` vs `ENTRYPOINT`?

-----

### Day 3: Docker Volumes - Persistent Data

**Concepts:**

  * **Ephemeral Nature of Containers:** By default, data written inside a container is lost when the container is removed.
  * **Volumes:** The preferred mechanism for persisting data generated by and used by Docker containers. They are managed by Docker and are more efficient than bind mounts for many use cases.
      * **Named Volumes:** Managed by Docker, created and referenced by name. Ideal for general-purpose persistence.
      * **Anonymous Volumes:** Also managed by Docker, but not given an explicit name. Less common.
  * **Bind Mounts:** You can mount a host machine's directory or file directly into a container. This is useful for development when you want to quickly see changes without rebuilding the image.

**Examples:**

1.  **Create a named volume:**
    ```bash
    docker volume create my-app-data
    ```
2.  **Inspect a volume:**
    ```bash
    docker volume inspect my-app-data
    ```
3.  **Run a container and attach the named volume:**
    ```bash
    docker run -d --name data-writer -v my-app-data:/app/logs alpine:latest sh -c "while true; do echo 'Current time: $(date)' >> /app/logs/access.log; sleep 1; done"
    ```
    (This runs an Alpine container in detached mode, names it `data-writer`, mounts `my-app-data` volume to `/app/logs`, and continuously writes to `access.log`.)
4.  **Inspect the volume data (from within a temporary container):**
    ```bash
    docker run --rm -v my-app-data:/data alpine:latest cat /data/access.log
    ```
    You can stop and remove `data-writer`, then re-run it, and the `access.log` content will persist because it's stored in the `my-app-data` volume.
5.  **Using a bind mount for development (live code changes):**
    Navigate to your `my-node-app` directory from Day 2.
    ```bash
    docker run -p 8000:3000 -v "$(pwd):/usr/src/app" my-node-app:1.0
    ```
    (Now, if you change `app.js` on your host, the changes will be reflected inside the running container instantly. For Node.js, you'd typically use `nodemon` inside the container for auto-restarts on file changes, but for this example, a manual restart of the container would show the change.)

**Challenge 3: Persistent Database**

  * Run a PostgreSQL or MySQL database in a Docker container.
  * Use a **named Docker volume** to ensure that your database data persists even if the container is removed or restarted.
  * Connect to the database from your host (e.g., using `psql` or `mysql` client within the container, or a GUI tool if you expose the port). Create a simple table and insert some data.
  * Remove the database container and then re-create it using the same volume. Verify that your data is still present.

-----

### Day 4: Docker Networking - Fundamentals & Host Communication

**Concepts:**

  * **Container Networking:** How Docker containers communicate with each other and with the outside world.
  * **Isolated Network Stack:** Each Docker container by default gets its own isolated network stack (IP address, routing table, DNS).
  * **Bridge Network (Default):** The most common network type. Containers on the same bridge network can communicate by IP address. Docker automatically creates a default `bridge` network.
  * **Host Network:** Container shares the host's network stack. No network isolation between the container and the host.
  * **None Network:** Disables all networking for the container.
  * **Port Mapping (`-p` or `--publish`):** The mechanism to map a port from the container's network stack to a port on the Docker host's network stack, allowing external access.

**Examples:**

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

**Challenge 4: Port Mapping & Container Reachability**

  * Run your Flask application from Day 2 in a container, but this time, map its internal port (e.g., 5000) to two different host ports (e.g., 8081 and 8082). Can you access it from both host ports?
  * Run two separate Nginx containers, both exposing port 80 internally, but map them to different host ports (e.g., 8083 and 8084). Verify you can reach both.
  * Try to run a container using `--network host` that starts a web server on port 80. What happens if your host machine already has a web server running on port 80?

-----

### Day 5: Docker Networking - User-Defined Bridge Networks & DNS

**Concepts:**

  * **Limitations of Default Bridge:** Containers on the default bridge can't communicate by name (only IP), and it's less secure/organized for multi-container apps.
  * **User-Defined Bridge Networks:** Best practice for multi-container applications.
      * **Automatic DNS Resolution:** Containers connected to the same user-defined bridge network can resolve each other by their container names.
      * **Better Isolation:** Services are isolated from the default bridge network.
      * **Portability:** Network configuration is part of the application definition (especially with Compose).

**Examples:**

1.  **Create a custom bridge network:**
    ```bash
    docker network create my-app-network
    ```
2.  **Inspect the custom network:**
    ```bash
    docker network inspect my-app-network
    ```
3.  **Run two containers on the custom network:**
    ```bash
    docker run -d --name web_server --network my-app-network nginx:latest
    docker run -d --name app_backend --network my-app-network alpine:latest sh -c "while true; do echo 'Backend is alive!'; sleep 5; done"
    ```
4.  **Test communication by container name:**
    ```bash
    docker exec -it web_server ping app_backend
    docker exec -it app_backend ping web_server
    ```
    (You should see successful pings because they are on the same user-defined network and can resolve each other by name.)
5.  **Connect an existing container to a network:**
    ```bash
    docker network connect my-app-network nginx_internal # (Use nginx_internal from Day 4 if it's still stopped)
    docker inspect nginx_internal | grep "Networks" # Verify it's now on two networks
    ```
6.  **Disconnect a container from a network:**
    ```bash
    docker network disconnect my-app-network nginx_internal
    ```
7.  **Remove the custom network:**
    ```bash
    docker network rm my-app-network
    ```
    (You need to stop/remove all containers connected to it first.)

**Challenge 5: Multi-Container Communication**

  * Run a simple "backend" container (e.g., using `alpine` with `netcat` to listen on a port, or a tiny Flask app) on a user-defined network.
  * Run a "frontend" container (e.g., Nginx) on the *same* user-defined network.
  * From the "frontend" container, try to `ping` the "backend" container using its container name.
  * Bonus: If your backend is a Flask app, configure Nginx to reverse proxy requests to the Flask app, using the Flask app's container name and internal port.

-----

### Day 6: Docker Networking - Advanced Concepts (Conceptual)

**Concepts:**

  * **Overlay Networks:** For connecting Docker daemons across multiple hosts. Essential for Docker Swarm and Kubernetes (though Kubernetes uses its own CNI plugins). Allows containers on different physical machines to communicate as if they were on the same local network.
  * **Macvlan Networks:** Assigns a MAC address to a container, making it appear as a physical device on your network. Useful for legacy applications that expect to directly control their network interface or for integrating with existing VLANs.
  * **Network Drivers:** Different network drivers provide different functionalities (e.g., `bridge`, `host`, `overlay`, `macvlan`).
  * **Network Isolation & Security:** Best practices for segmenting applications into different networks.

**Examples (Conceptual - no complex setup required, focus on understanding):**

1.  **Discuss Overlay Networks:** Explain how they enable multi-host container communication. Mention they require a key-value store (like Consul or etcd) or a Swarm mode.
2.  **Discuss Macvlan Networks:** Explain how they give containers a unique MAC address and appear as first-class citizens on the physical network.
3.  **Use Cases:** When would you use an overlay network (e.g., a distributed microservice architecture)? When would you use a Macvlan network (e.g., direct IP access, IoT devices)?
4.  **Network Policies (Brief Mention):** For more granular control over container-to-container communication within a network (often handled by orchestration tools).

**Challenge 6: Network Driver Research**

  * Research a real-world scenario where an **Overlay Network** would be indispensable. Describe the problem it solves.
  * Research a real-world scenario where a **Macvlan Network** would be particularly useful. Describe why it's a good fit.
  * What are the security implications of using `--network host` in production?

-----

### Day 7: Docker Compose - Introduction & Basic Services

**Concepts:**

  * **Docker Compose:** A tool for defining and running multi-container Docker applications. You define your application's services, networks, and volumes in a single `docker-compose.yml` file.
  * **`docker-compose.yml`:** The core configuration file for Compose, written in YAML.
  * **Services:** Individual containers that make up your application. Each service corresponds to a container that will be run.
  * **`up` and `down` commands:** How to start and stop your entire application stack.
  * **Automatic Network Creation:** Compose automatically creates a default user-defined bridge network for all services defined in the `docker-compose.yml` file, allowing them to communicate by service name.
  * **Port Mapping in Compose:** Defining host-to-container port mappings in the `yml` file.

**Examples:**

1.  **Create a new directory for your Compose project:** `my-first-compose-app`.
2.  **`app.py` (Simple Flask app for web service):**
    ```python
    from flask import Flask
    from redis import Redis
    import os

    app = Flask(__name__)
    # 'redis' will be the hostname for the Redis service as defined in docker-compose.yml
    redis_host = os.getenv('REDIS_HOST', 'redis')
    redis = Redis(host=redis_host, port=6379) 

    @app.route('/')
    def hello():
        try:
            count = redis.incr('hits')
            return f'Hello from Docker! I have been seen {count} times.\n'
        except Exception as e:
            return f"Error connecting to Redis: {e}\n", 500

    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000, debug=True)
    ```
3.  **`requirements.txt`:**
    ```
    Flask
    redis
    ```
4.  **`Dockerfile` (for the Flask app - placed in `my-first-compose-app`):**
    ```dockerfile
    FROM python:3.9-slim-buster
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install -r requirements.txt
    COPY . .
    EXPOSE 5000
    CMD ["python", "app.py"]
    ```
5.  **`docker-compose.yml` (in `my-first-compose-app` directory):**
    ```yaml
    version: '3.8' # Specify Compose file format version

    services:
      web: # Define a service named 'web'
        build: . # Build the image from the Dockerfile in the current directory
        ports:
          - "8000:5000" # Map host port 8000 to container port 5000
        environment:
          REDIS_HOST: redis # Pass environment variable to the container
        depends_on: # Ensure redis service starts before web (does not wait for readiness)
          - redis 
      redis: # Define a service named 'redis'
        image: "redis:alpine" # Use the official Redis Alpine image from Docker Hub
    ```
6.  **Start the application:**
    Navigate to the `my-first-compose-app` directory in your terminal.
    ```bash
    docker-compose up -d
    ```
    (The `-d` flag runs containers in detached mode.)
7.  **Access the application:** Open `http://localhost:8000` in your browser and refresh to see the counter increase.
8.  **View logs for all services or specific services:**
    ```bash
    docker-compose logs
    docker-compose logs web
    docker-compose logs redis
    ```
9.  **Check running services:**
    ```bash
    docker-compose ps
    ```
10. **Stop and remove containers and network created by Compose:**
    ```bash
    docker-compose down
    ```

**Challenge 7: Compose a Simple Nginx and Static Site**

  * Create a new directory for this challenge.
  * Create a simple `index.html` file (e.g., "Hello from Nginx\!").
  * Write a `docker-compose.yml` file to:
      * Define a service for Nginx using the `nginx:latest` image.
      * Map a host port (e.g., 8080) to Nginx's default web port (80).
      * Use a **bind mount** to serve your `index.html` from your host machine into the Nginx container's web root (`/usr/share/nginx/html`).
  * Bring up the service with `docker-compose up -d` and verify you can access your static page.

-----

### Day 8: Docker Compose - Networking & Volumes in Depth

**Concepts:**

  * **User-defined Networks in Compose (Explicitly):** How to define and connect services to custom networks within your `docker-compose.yml` for better organization and explicit control.
  * **Volumes in Compose:**
      * **Named Volumes:** Defining and attaching named volumes directly in `docker-compose.yml` for persistent data.
      * **Bind Mounts:** Using bind mounts for development workflows.
  * **Volume Drivers:** Briefly mention that volumes can use different drivers for specific storage needs (e.g., for cloud storage).

**Examples:**

1.  **Explicit Custom Networks in Compose:**
    Let's modify the `docker-compose.yml` from Day 7 to explicitly define the network:
    ```yaml
    version: '3.8'

    services:
      web:
        build: .
        ports:
          - "8000:5000"
        environment:
          REDIS_HOST: redis
        networks: # Connect 'web' service to 'app_network'
          - app_network 
        depends_on:
          - redis 
      redis:
        image: "redis:alpine"
        volumes:
          - redis_data:/data # Attach the named volume 'redis_data'
        networks: # Connect 'redis' service to 'app_network'
          - app_network

    volumes: # Define the named volume (managed by Docker)
      redis_data:

    networks: # Define the custom network (managed by Compose within this project)
      app_network:
        driver: bridge # Explicitly set driver (default is bridge)
    ```
    Bring it up (`docker-compose up -d`). `docker network ls` will show a network like `my-first-compose-app_app_network`. `docker network inspect` will show both containers connected.
2.  **Bind Mount for persistent development (e.g., for your Flask app from Day 7's challenge):**
    If you had your Flask app in a `backend/` subdirectory:
    ```yaml
    version: '3.8'
    services:
      web:
        build: ./backend # Build from backend directory
        ports:
          - "8000:5000"
        volumes:
          - ./backend:/app # Mount host's backend directory to container's /app
    ```
    This allows live code changes during development without rebuilding the image.
3.  **Using `tmpfs` mounts (for non-persistent, high-performance needs):**
    ```yaml
    services:
      temp_processor:
        image: alpine
        tmpfs: /tmp/data:size=100m # Mount a tmpfs at /tmp/data with 100MB limit
    ```

**Challenge 8: Multi-Tiered Application with Explicit Networking and Persistence**

  * Create a `docker-compose.yml` for a simple application with:
      * An Nginx web server (frontend).
      * Your Flask application (backend) that connects to a PostgreSQL database.
      * A PostgreSQL database.
  * Define **two distinct user-defined networks**:
      * `web_network`: Connects Nginx and your Flask backend.
      * `db_network`: Connects your Flask backend and PostgreSQL.
      * Your Flask backend should be connected to *both* networks.
  * Use a **named volume** for PostgreSQL's data persistence.
  * Ensure that Nginx can forward requests to your Flask app via its service name, and Flask can connect to PostgreSQL via its service name.
  * Bring up the stack and verify all connections work.

-----

### Day 9: Docker Compose - Advanced Features & Best Practices

**Concepts:**

  * **`depends_on` revisited:** Understanding its limitations (only ensures start order, not service readiness).
  * **Healthchecks:** Defining commands that Docker can run periodically to check if a service is healthy, allowing Compose to wait for services to be truly ready.
  * **Environment Variables:** Passing configuration to services (using `environment` and `.env` files).
  * **Building Images with Compose:** The `build` key for building images on the fly.
  * **Scaling Services:** Using `docker-compose up --scale` to run multiple instances of a service.
  * **`profiles` (Compose v3.4+):** For running different subsets of services in development vs. testing vs. production.
  * **`extends` (Compose v2/3):** Reusing common configurations across multiple Compose files.

**Examples:**

1.  **Healthchecks (improving `depends_on`):**
    ```yaml
    version: '3.8'
    services:
      web:
        build: .
        ports:
          - "8000:5000"
        environment:
          REDIS_HOST: redis
        depends_on:
          redis:
            condition: service_healthy # Wait for redis to be healthy
      redis:
        image: "redis:alpine"
        healthcheck: # Define a healthcheck for redis
          test: ["CMD", "redis-cli", "ping"]
          interval: 1s
          timeout: 3s
          retries: 5
    ```
2.  **Using `.env` files for environment variables:**
    Create a file named `.env` in the same directory as `docker-compose.yml`:
    ```
    DB_USER=myuser
    DB_PASSWORD=mypassword
    ```
    Then, in `docker-compose.yml`:
    ```yaml
    version: '3.8'
    services:
      db:
        image: postgres:13
        environment:
          POSTGRES_USER: ${DB_USER} # Uses variable from .env
          POSTGRES_PASSWORD: ${DB_PASSWORD} # Uses variable from .env
    ```
    Docker Compose will automatically load variables from a `.env` file.
3.  **Scaling a service:**
    ```bash
    docker-compose up -d --scale web=3 # Runs 3 instances of the 'web' service
    docker-compose ps # Observe multiple web containers
    ```
    Then scale back down:
    ```bash
    docker-compose up -d --scale web=1
    ```
4.  **`profiles` for development vs. production services:**
    ```yaml
    version: '3.8'
    services:
      app:
        build: .
        profiles: ["app"] # Only starts with 'app' profile
      db:
        image: postgres
        profiles: ["app"]
      test_runner:
        image: my-test-image
        profiles: ["test"] # Only starts with 'test' profile
    ```
    Run with specific profile: `docker-compose --profile app up -d`

**Challenge 9: Robust Application with Healthchecks and Secrets**

  * Take your multi-tiered application from Challenge 8.
  * Add a **healthcheck** to your PostgreSQL service to ensure it's truly ready before your Flask app attempts to connect.
  * Use a `.env` file to manage sensitive information like database credentials, rather than hardcoding them in `docker-compose.yml`.
  * Experiment with scaling your web service using `docker-compose up --scale`. Observe how Docker handles the multiple instances.

-----

### Day 10: Docker Image Optimization & Best Practices

**Concepts:**

  * **Multi-stage Builds:** Optimize Docker images by using multiple `FROM` instructions in a Dockerfile. This allows you to use a larger base image with build tools for compilation, and then copy only the necessary artifacts to a smaller, final runtime image.
  * **`.dockerignore`:** Similar to `.gitignore`, a `.dockerignore` file specifies files and directories to exclude from the build context, reducing image size and build time.
  * **Image Optimization Best Practices:** Strategies to create smaller, more secure, and faster Docker images (e.g., using official base images, minimizing layers, removing unnecessary packages, using smaller base images like Alpine).
  * **Security Considerations:** Running containers as non-root users, regularly updating base images, scanning images for vulnerabilities (brief mention of tools).
  * **Cleaning Up Docker Resources:** Commands to remove unused images, containers, volumes, and networks.

**Examples:**

1.  **Multi-stage build for a Node.js app (revisit and optimize `my-node-app`):**
    ```dockerfile
    # Stage 1: Build dependencies and potentially a frontend build
    FROM node:18-alpine as builder
    WORKDIR /usr/src/app
    COPY package*.json ./
    RUN npm install --production # Install only production dependencies
    # If you had a build step for a frontend framework like React/Vue:
    # COPY frontend ./frontend
    # RUN npm run build

    # Stage 2: Create the final lean image
    FROM node:18-alpine
    WORKDIR /usr/src/app
    # Copy only the necessary files from the builder stage
    COPY --from=builder /usr/src/app/node_modules ./node_modules
    COPY --from=builder /usr/src/app/app.js .
    COPY --from=builder /usr/src/app/package.json . # Needed for `npm start`
    # If you had a build step:
    # COPY --from=builder /usr/src/app/build ./build

    # Best practice: Run as a non-root user
    RUN addgroup -S appgroup && adduser -S appuser -G appgroup
    USER appuser

    EXPOSE 3000
    CMD [ "npm", "start" ]
    ```
    Build and run:
    ```bash
    docker build -t my-node-app-optimized -f Dockerfile.multistage .
    docker run -p 8000:3000 my-node-app-optimized
    ```
    Compare the size using `docker images`.
2.  **Using `.dockerignore`:**
    In your project root, create a `.dockerignore` file:
    ```
    node_modules/
    .git/
    npm-debug.log
    *.swp
    tmp/
    **/node_modules/
    ```
    This prevents unnecessary files from being sent to the Docker daemon during build.
3.  **Cleaning up:**
    ```bash
    docker system prune          # Removes all stopped containers, all dangling images, and all unused networks
    docker system prune -a       # Removes all unused containers, networks, images (both dangling and unreferenced), and optionally volumes. Use with caution!
    docker volume rm <volume_name> # Remove a specific volume
    docker network rm <network_name> # Remove a specific network
    docker image prune -a        # Removes all unused images
    ```

**Challenge 10: Optimize Your App Image**

  * Take your Flask application's `Dockerfile` from previous days.
  * Implement a **multi-stage build** for it. For a Python app, this often means installing dependencies in one stage and then copying only the application code and the virtual environment (or site-packages) to a smaller runtime image.
  * Create a comprehensive **`.dockerignore`** file for your Flask project.
  * If possible, modify your Dockerfile to run the application as a **non-root user**.
  * Compare the size of the original image with the optimized one using `docker images`.

-----

### Day 11: Docker Registries & Distribution

**Concepts:**

  * **Image Registries:** Centralized repositories for storing and sharing Docker images (e.g., Docker Hub, Google Container Registry, AWS ECR, GitLab Container Registry, private registries).
  * **`docker login`:** Authenticating with a registry.
  * **`docker tag`:** Tagging images with repository names and versions for pushing.
  * **`docker push`:** Uploading images to a registry.
  * **`docker pull`:** Downloading images from a registry.
  * **Image Security:** Brief mention of image scanning, content trust (Docker Notary).

**Examples:**

1.  **Login to Docker Hub:**
    ```bash
    docker login
    ```
    (Enter your Docker Hub username and password.)
2.  **Tag your optimized image from Day 10:**
    ```bash
    docker tag my-flask-app-optimized <your_dockerhub_username>/my-flask-app:1.0
    ```
    (Replace `<your_dockerhub_username>` with your actual Docker Hub username.)
3.  **Push the image to Docker Hub:**
    ```bash
    docker push <your_dockerhub_username>/my-flask-app:1.0
    ```
    (This might take some time depending on your internet connection and image size.)
4.  **Verify on Docker Hub website:** Log in to Docker Hub in your browser and confirm that your repository and image are visible.
5.  **Pull your image from Docker Hub (simulate on a new machine or after local removal):**
    ```bash
    docker rmi <your_dockerhub_username>/my-flask-app:1.0 # Remove local copy first
    docker pull <your_dockerhub_username>/my-flask-app:1.0
    docker run -p 8080:5000 <your_dockerhub_username>/my-flask-app:1.0
    ```
    (Access `http://localhost:8080` to verify.)

**Challenge 11: Share Your Work**

  * Push your optimized Flask application image from Day 10 to your Docker Hub account with a new tag (e.g., `v2.0`).
  * In a separate terminal or on another machine if possible, pull this new image.
  * Run a container from the pulled image and verify it works as expected.
  * (Optional) If you have a friend learning Docker, share your image name and ask them to try pulling and running it\!

-----

### Day 12: Docker in Practice - CI/CD & Orchestration (Conceptual & Next Steps)

**Concepts:**

  * **Docker in CI/CD Workflows:** How Docker fits into automated build, test, and deployment pipelines. (Build image -\> Test in container -\> Push image -\> Deploy container).
  * **Container Orchestration:** Tools for managing and scaling containerized applications in production environments.
      * **Docker Swarm:** Docker's native orchestration tool, simpler to set up for smaller scale.
      * **Kubernetes:** The industry standard for large-scale, complex container orchestration.
  * **Microservices Architectures:** How Docker facilitates breaking down applications into smaller, independent services.
  * **Serverless (Brief Mention):** How containerization powers some serverless platforms.

**Examples (Conceptual - no hands-on required, focus on understanding):**

1.  **CI/CD Pipeline Sketch:**
      * Developer pushes code to Git.
      * CI system (Jenkins, GitLab CI, GitHub Actions) triggers:
          * `docker build -t my-app:$(GIT_COMMIT_SHA) .`
          * `docker run my-app:$(GIT_COMMIT_SHA) npm test` (Run tests inside container)
          * If tests pass: `docker push my-app:$(GIT_COMMIT_SHA)`
      * CD system deploys the new image to production (using Swarm, Kubernetes, etc.).
2.  **Introduction to Orchestration Needs:** Discuss why `docker-compose` is insufficient for production (e.g., single point of failure, manual scaling, no self-healing, no rolling updates).
3.  **Kubernetes vs. Docker Swarm:** Briefly compare their complexity, features, community support, and typical use cases.
      * **Swarm:** Simpler, good for Docker-centric ecosystems, smaller deployments.
      * **Kubernetes:** Powerful, complex, highly extensible, industry standard for large-scale deployments, cloud-agnostic.
4.  **Benefits of Microservices with Docker:** Independent development, deployment, scaling, fault isolation.

**Challenge 12: Your Docker Journey - Reflection & Future Steps**

  * **Reflection:** What were the most challenging aspects of this tutorial for you? What concepts clicked well?
  * **Future Steps (Research):**
      * Choose either Docker Swarm or Kubernetes. Research its basic architecture (e.g., master/worker nodes, pods, deployments, services in Kubernetes).
      * Find a simple "getting started" tutorial for your chosen orchestration tool (e.g., Minikube for Kubernetes, `docker swarm init` for Swarm). Don't run it now, just identify the steps.
      * How would your blog application (from Challenge 9) be deployed and managed using your chosen orchestration tool? Sketch out the high-level steps.
