# Day 11: Docker Registries & Distribution

## **Concepts:**

  * **Image Registries:** Centralized repositories for storing and sharing Docker images (e.g., Docker Hub, Google Container Registry, AWS ECR, GitLab Container Registry, private registries).
  * **`docker login`:** Authenticating with a registry.
  * **`docker tag`:** Tagging images with repository names and versions for pushing.
  * **`docker push`:** Uploading images to a registry.
  * **`docker pull`:** Downloading images from a registry.
  * **Image Security:** Brief mention of image scanning, content trust (Docker Notary).

## **Examples:**

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

