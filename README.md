# Demo Projects: Containers with Docker

## Project 1: Use Docker for Local Development

### Technologies Used
- **Docker**
- **Node.js**
- **MongoDB**
- **MongoExpress**

### Project Description
- Created a **Dockerfile** for a Node.js application and built a Docker image.
- Ran the Node.js application in a Docker container and connected it to a MongoDB database container locally.
- Deployed **MongoExpress** as a container to serve as the UI for the MongoDB database.

---

## Project 2: Docker Compose - Run Multiple Docker Containers

### Technologies Used
- **Docker**
- **MongoDB**
- **MongoExpress**

### Project Description
- Wrote a **Docker Compose** file to manage and run multiple Docker containers for:
  - **MongoDB**: Database service.
  - **MongoExpress**: UI for MongoDB database management.

---

## Project 3: Deploy Docker Application on a Server with Docker Compose

### Technologies Used
- **Docker**
- **Amazon ECR**
- **Node.js**
- **MongoDB**
- **MongoExpress**

### Project Description
- Copied the **Docker Compose** file to a remote server.
- Logged in to a private Docker registry on the remote server to fetch the application image.
- Deployed and started the application container along with **MongoDB** and **MongoExpress** services using Docker Compose.

---

## Project 4: Dockerize Node.js Application and Push to Private Docker Registry

### Technologies Used
- **Docker**
- **Node.js**
- **Amazon ECR**

### Project Description
- Wrote a **Dockerfile** to build a Docker image for a Node.js application.
- Created a private Docker registry on **AWS (Amazon ECR)**.
- Pushed the Docker image to this private repository for secure storage and deployment.

---

## Project 5: Create Docker Repository on Nexus and Push to It

### Technologies Used
- **Docker**
- **Nexus**
- **DigitalOcean**
- **Linux**

### Project Description
- Created a Docker-hosted repository on **Nexus**.
- Configured **Nexus**, a DigitalOcean Droplet, and Docker to push images to the Nexus repository.
- Built and pushed a Docker image to the Docker repository hosted on Nexus.

---

## Project 6: Persist Data with Docker Volumes

### Technologies Used
- **Docker**
- **Node.js**
- **MongoDB**

### Project Description
- Configured a **Docker Volume** to persist data for a MongoDB container.
- Ensured data durability and availability across container restarts.

---

## Project 7: Deploy Nexus as Docker Container

### Technologies Used
- **Docker**
- **Nexus**
- **DigitalOcean**
- **Linux**

### Project Description
- Created and configured a Droplet on **DigitalOcean**.
- Set up and deployed **Nexus** as a Docker container for repository management.

---

# Demo Projects: Build Automation & CI/CD with Jenkins

## Project 1: Install Jenkins on DigitalOcean

### Technologies Used
- **Jenkins**
- **Docker**
- **DigitalOcean**
- **Linux**

### Project Description
- Created an **Ubuntu server** on **DigitalOcean**.
- Set up and ran **Jenkins** as a Docker container.
- Initialized Jenkins for use in build automation.

---

## Project 2: Create a CI Pipeline with Jenkinsfile (Freestyle, Pipeline, Multibranch Pipeline)

### Technologies Used
- **Jenkins**
- **Docker**
- **Linux**
- **Git**
- **Java**
- **Maven**

### Project Description
- Built a CI Pipeline for a **Java Maven application** to build and push artifacts to the repository.
- Installed build tools (Maven, Node.js) in Jenkins.
- Made Docker available on the Jenkins server.
- Created Jenkins credentials for a Git repository.
- Set up different Jenkins job types (Freestyle, Pipeline, Multibranch Pipeline) for:
  - Connecting to the application’s Git repository.
  - Building JAR files.
  - Building Docker images.
  - Pushing images to a private **DockerHub** repository.

---

## Project 3: Create a Jenkins Shared Library

### Technologies Used
- **Jenkins**
- **Groovy**
- **Docker**
- **Git**
- **Java**
- **Maven**

### Project Description
- Created a **Jenkins Shared Library (JSL)** to extract common build logic.
- Developed a separate Git repository for the JSL project.
- Wrote reusable functions in the JSL for use in Jenkins pipelines.
- Integrated and utilized the JSL in Jenkins pipelines globally and for specific projects in **Jenkinsfiles**.

---

## Project 4: Configure Webhook to Trigger CI Pipeline Automatically on Every Change

### Technologies Used
- **Jenkins**
- **GitLab**
- **Git**
- **Docker**
- **Java**
- **Maven**

### Project Description
- Installed the **GitLab Plugin** in Jenkins.
- Configured a **GitLab access token** and established a connection to Jenkins in GitLab project settings.
- Set up Jenkins to trigger the CI pipeline automatically whenever a change was pushed to GitLab.

---

## Project 5: Dynamically Increment Application Version in Jenkins Pipeline

### Technologies Used
- **Jenkins**
- **Docker**
- **GitLab**
- **Git**
- **Java**
- **Maven**

### Project Description
- Configured the CI pipeline to:
  - Increment the application patch version dynamically.
  - Build the Java application and clean old artifacts.
  - Build a Docker image with a dynamic Docker image tag.
  - Push the image to a private **DockerHub** repository.
  - Commit the version update back to the Git repository.
- Ensured the Jenkins pipeline did not trigger automatically on CI build commits to avoid commit loops.
