# Day 10: Docker Image Optimization & Best Practices

## **Concepts:**

  * **Multi-stage Builds:** Optimize Docker images by using multiple `FROM` instructions in a Dockerfile. This allows you to use a larger base image with build tools for compilation, and then copy only the necessary artifacts to a smaller, final runtime image.
  * **`.dockerignore`:** Similar to `.gitignore`, a `.dockerignore` file specifies files and directories to exclude from the build context, reducing image size and build time.
  * **Image Optimization Best Practices:** Strategies to create smaller, more secure, and faster Docker images (e.g., using official base images, minimizing layers, removing unnecessary packages, using smaller base images like Alpine).
  * **Security Considerations:** Running containers as non-root users, regularly updating base images, scanning images for vulnerabilities (brief mention of tools).
  * **Cleaning Up Docker Resources:** Commands to remove unused images, containers, volumes, and networks.

## **Examples:**

1.  **Multi-stage build for a Node.js app (revisit and optimize `my-node-app`):**
```dockerfile
# Stage 1: Cache Node.js dependencies
FROM node:18-alpine as builder

WORKDIR /usr/src/app

# Copy package.json and package-lock.json to leverage Docker's build cache
# This layer will only be rebuilt if package*.json changes.
COPY package*.json ./

# Install all dependencies (including devDependencies) in this caching stage.
# This ensures that if you had a build step (e.g., for a frontend),
# all necessary tools would be available.
RUN npm install

# Stage 2: Create the final lean production image
FROM node:18-alpine

WORKDIR /usr/src/app

# Copy package.json and package-lock.json from the build context (your local directory).
# This is crucial for `npm ci` in the next step, as it strictly requires a lockfile.
COPY package*.json ./

# Copy the rest of your application code from the build context.
# This includes app.js and any other source files.
COPY . .

# Install only production dependencies using `npm ci`.
# This command is designed for clean installs in CI/CD environments
# and will create the node_modules directory directly in this stage.
# `--omit=dev` is preferred over `--production` for newer npm versions.
RUN npm install --production 

# Best practice: Run as a non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000

# Define the command to run the application when the container starts
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

## **Challenge 10: Optimize Your App Image**

  * Take your Flask application's `Dockerfile` from previous days.
  * Implement a **multi-stage build** for it. For a Python app, this often means installing dependencies in one stage and then copying only the application code and the virtual environment (or site-packages) to a smaller runtime image.
  * Create a comprehensive **`.dockerignore`** file for your Flask project.
  * If possible, modify your Dockerfile to run the application as a **non-root user**.
  * Compare the size of the original image with the optimized one using `docker images`.

