# Day 8: Docker Compose - Networking & Volumes in Depth

## **Concepts:**

  * **User-defined Networks in Compose (Explicitly):** How to define and connect services to custom networks within your `docker-compose.yml` for better organization and explicit control.
  * **Volumes in Compose:**
      * **Named Volumes:** Defining and attaching named volumes directly in `docker-compose.yml` for persistent data.
      * **Bind Mounts:** Using bind mounts for development workflows.
  * **Volume Drivers:** Briefly mention that volumes can use different drivers for specific storage needs (e.g., for cloud storage).

## **Examples:**

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

## **Challenge 8: Multi-Tiered Application with Explicit Networking and Persistence**

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
