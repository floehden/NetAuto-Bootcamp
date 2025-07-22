# Day 9: Docker Compose - Advanced Features & Best Practices

## **Concepts:**

  * **`depends_on` revisited:** Understanding its limitations (only ensures start order, not service readiness).
  * **Healthchecks:** Defining commands that Docker can run periodically to check if a service is healthy, allowing Compose to wait for services to be truly ready.
  * **Environment Variables:** Passing configuration to services (using `environment` and `.env` files).
  * **Building Images with Compose:** The `build` key for building images on the fly.
  * **Scaling Services:** Using `docker-compose up --scale` to run multiple instances of a service.
  * **`profiles` (Compose v3.4+):** For running different subsets of services in development vs. testing vs. production.
  * **`extends` (Compose v2/3):** Reusing common configurations across multiple Compose files.

## **Examples:**

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

## **Challenge 9: Robust Application with Healthchecks and Secrets**

  * Take your multi-tiered application from Challenge 8.
  * Add a **healthcheck** to your PostgreSQL service to ensure it's truly ready before your Flask app attempts to connect.
  * Use a `.env` file to manage sensitive information like database credentials, rather than hardcoding them in `docker-compose.yml`.
  * Experiment with scaling your web service using `docker-compose up --scale`. Observe how Docker handles the multiple instances.

