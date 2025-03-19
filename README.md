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

### Practice 1 : Create Ubuntu Server on DigitalOcean 

  - Step 1 : Go to Digital ocean -> Create Droplet -> Choose Region and Capacity -> Create SSH key

  - Step 2 : To create SSH key :

    - In terminal : `ssh-keygen` . There will be .ssh/id_rsa and .ssh/id_rsa.pub . Then `cat .ssh/id_rsa.pub` take this content and put it into Digital ocean
   
    <img width="600" alt="Screenshot 2025-03-18 at 14 37 03" src="https://github.com/user-attachments/assets/49d7f9df-e2c3-4162-9a5e-7495435fd593" />

### Practice 2 : Set up and ran Jenkins as Docker container 

  - Step 1 : Connect to Server : `ssh root@<IP-address>`

  - Step 2 : Create Firewall Rule to it

      - Add Custom Port 8080 : This is where Jenkin start . This is where I will expose it 
   
      - Add Port 22 : To SSH
   
  - Step 3 : Install Docker : `apt install docker.io`

  - Step 4 : Install Jenkins : `docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts`

    - First 8080 Port : Where the browser access to (The server itself).
   
    - Second 8080 Port : Is the Port of the Container itsefl (Jenkins itself) 
   
    - Port 50000:50000 : This is where Jenkins Master and Worker nodes communicate . Jenkins can actually built and started as a Cluster if I have large Workloads that I am running with Jenkins
   
    - -d : Detach Mode. Run the cotainer in the background
   
    - -v jenkins_home:/var/jenkins_home: Mount volumes
        
        - Jenkins is just like Nexus, It will store a lot of data . When I configure Jenkins, Create User, Create Jobs to run, Install Plugin and so ons . All of these will be store as Data
     
        - jenkins_home : This folder doesn't exist yet (Name Volume references) . Docker will create a physical path on the server will store a data with that Name References
     
        - /var/jenkins_home : This is a Actual directory in Cotnainer (Inside Jenkins) that will store data
     
  - Step 5 : In the UI . First Access Jenkins will give me a path to get the Password . I will `docker exec -t <container-id> bash` go inside Jenkins and get the password

  - To check Volume that I create : `docker inspect volume jenkins_home`

### Install Build tools in Jenkins

<img width="937" alt="Screenshot 2025-03-19 at 09 00 17" src="https://github.com/user-attachments/assets/03f85e65-b362-4bb2-8053-81d2ae3310d9" />

  - Depending on my Application, I have different tools installed and configured on Jenkins

**2 ways to install and configure those tools**

  1. Jenkins Plugin

    - Install plugin via UI for my tools 

    - In Jenkins UI -> go to Tools -> Choose tools and tools version that I want to intall -> Then I should have those tools available in Jobs 

  2. Install those tool directly on the Server

    - Go inside Jenkins `docker exec -it -u 0 <docker-id> bash` and install tools in there  

    - Look up what distribution of Linux my server is running so I can find respective installation guild for that tools : `cat etc/issue`

    - Update package Manager : `apt update`

    - Install curl : `apt install curl` . Curl is a command-line tool and library used for tranfering data with URL . Support HTTP, HTTPS, FTP, FTPS, SCP ... and more . 

    - Using curl to get a Script that and install Nodejs and Npm : `curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh` . Then execute that Script `bash nodesource_setup.sh` then `apt install nodejs`

  3. Install Stage View Plugin

    - This Plugins help me see diffent stage defined in the UI . This mean Build Stage, Test, Deploy will displayed as separate stage in the UI 

    - Go to Available Plugin -> Stage View

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

