# Day 2: Dockerfiles - Building Custom Images

## **Concepts:**

  * **Dockerfile:** A text file containing instructions for building a Docker image. Each instruction creates a layer in the image.
  * **Build Context:** The directory containing the Dockerfile and any files it needs to copy into the image.
  * **Layers & Caching:** Docker images are composed of layers, where each instruction in a Dockerfile adds a new layer. This allows for efficient caching during builds.
  * **Common Dockerfile Instructions:** `FROM`, `WORKDIR`, `COPY`, `ADD`, `RUN`, `CMD`, `ENTRYPOINT`, `EXPOSE`.

## **Examples:**

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

## **Challenge 2: Containerize a Python Flask App**

  * Create a simple Python Flask application that returns "Hello from Flask in Docker\!" on port 5000.
  * Write a `Dockerfile` to containerize this Flask application.
  * Build the image and run a container, ensuring you can access the Flask app from your host browser.
  * Experiment with `CMD` vs `ENTRYPOINT`. What happens if you try to `docker run <your_image_name> echo "Hello"` when using `CMD` vs `ENTRYPOINT`?
