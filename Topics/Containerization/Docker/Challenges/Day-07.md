# Day 7: Docker Compose - Introduction & Basic Services

## **Concepts:**

  * **Docker Compose:** A tool for defining and running multi-container Docker applications. You define your application's services, networks, and volumes in a single `docker-compose.yml` file.
  * **`docker-compose.yml`:** The core configuration file for Compose, written in YAML.
  * **Services:** Individual containers that make up your application. Each service corresponds to a container that will be run.
  * **`up` and `down` commands:** How to start and stop your entire application stack.
  * **Automatic Network Creation:** Compose automatically creates a default user-defined bridge network for all services defined in the `docker-compose.yml` file, allowing them to communicate by service name.
  * **Port Mapping in Compose:** Defining host-to-container port mappings in the `yml` file.

## **Examples:**

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
sudo docker-compose up -d
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

## **Challenge 7: Compose a Simple Nginx and Static Site**

  * Create a new directory for this challenge.
  * Create a simple `index.html` file (e.g., "Hello from Nginx\!").
  * Write a `docker-compose.yml` file to:
      * Define a service for Nginx using the `nginx:latest` image.
      * Map a host port (e.g., 8080) to Nginx's default web port (80).
      * Use a **bind mount** to serve your `index.html` from your host machine into the Nginx container's web root (`/usr/share/nginx/html`).
  * Bring up the service with `docker-compose up -d` and verify you can access your static page.

