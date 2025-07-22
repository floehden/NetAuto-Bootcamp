# Day 1: Docker Fundamentals - Images & Containers

## **Concepts:**

* **Images:** Read-only templates used to create containers. They contain the application code, libraries, dependencies, and configurations.
* **Containers:** Runnable instances of an image. They are isolated, lightweight environments where your application runs.
* **Docker Daemon:** The background service that manages Docker objects.
* **Docker Client:** The command-line tool (or other clients) that interacts with the Docker Daemon.
* **Docker Hub:** A public registry for Docker images.

## **Examples:**

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

## **Challenge 1: First Container Interaction**

* Run a `debian:latest` container in interactive mode and try to install a package like `curl` inside it. What happens and why? (Hint: Think about what's included in a minimal Debian image.)
* After exiting, try to start and re-attach to the same container.
* Clean up by removing the container.

