# Day 5: Docker Networking - User-Defined Bridge Networks & DNS

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
