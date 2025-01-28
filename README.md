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

## Project 2: Create a Jenkins Shared Library

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

## Project 3: Configure Webhook to Trigger CI Pipeline Automatically on Every Change

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

## Project 4: Dynamically Increment Application Version in Jenkins Pipeline

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

---

## Project 5: Create a CI Pipeline with Jenkinsfile (Freestyle, Pipeline, Multibranch Pipeline)

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

