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

### Practice 3:  Install Build tools in Jenkins

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

### Practice 4 : Docker in Jenkins 

  - Most of scenerio I will need to build Docker Image in Jenkins . That mean I need Docker Command in Jenkins . The way to do that is attaching a volume to Jenkins from the host file

  - In the Server (Droplet itself) I have Docker command available, I will mount Docker directory from Droplet into a Container as a volume . This will make Docker available inside the container

  - To do that I first need to kill current Container and create a new : `docker stop <container-id>`

  - Check the volume : `docker ls volume` . All the data from the container before will be persist in here and I can use that to create a new Container

  - Start a new container : `docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts`

    -  /var/run/docker.sock:/var/run/docker.sock : I mount a Docker from Droplet to inside Jenkins
   
  - Get inside Jenkins as Root : `docker exec -it -u 0 <container_id> bash`

    - Things need to fix :

      - `curl https://get.docker.com/ > dockerinstall && chmod 777 dockerinstall && ./dockerinstall` . With this Curl command Jenkins container is going to fetch the latest Version of Docker from official size so it can run inside the container, then I will set correct permission the run through the Install
      - Set correct Permission on `docker.sock` so I can run command inside the container as Jenkins User  `chmod 666 /var/run/docker.sock`: docker.sock is a Unix socket file used by Docker daemon to communicate with Docker Client
     
### Pipeline Jobs 

  - Stuitable for CI/CD

  - Scripting - Pipeline as Code

**Connect Pipeline build to Git Repo**

  - The main use case for all the builds that I need to execute workflow for my Application Repo

**Pipeline**

  - This part is what I will do using Script

  - There is 2 options : Pipline Script, Pipeline Script from Source Code Management

  - Script is written in Groovy

  - Groovy Sandbox: I am allow to use limited number of Groovy built-in functions in this Script without needing Permission from Jenkins Administator . So these are the Functions does not seem as risky or insecure when they are executed on Jenkins they are kind of Whitelisted and If I want to use Groovy Library or some other Function then for those specific Function I would need Approval of Jenkins Admin

  - However The best practice is to have infrastructure, the configuration management and all this configuration in my Application code . Which mean I should have Groovy script in my Source Code Management together with my Application .

**Source Code Management Script in Pipeline Set up**

  - Repository URL : My github repo URL

  - Credentials : Credentials of my Github

  - Branch to build : */<branch name> : If I leave it as * It will build the whole Branches .

  - This mean this Pipeline Job will check out the Repository and inside that Repository in the Root Folder it will look for a file call Jenkinsfile

**Jenkins File**

  - Jenkins file can be written as a scripted Pipeline or Declarative Pipeline .

  - Scripted : First Syntax, Groovy Engine , Advanced scripting capabilities high flexibilities .

  - Declarative : More recent addition, easier to start bcs I have predefined Structure

  - Required field of Jenkins :

    - pipeline : Must be top level
   
    - agent any: This build on any available Jenkins Agent . Agent can be a Node, it could be executable on that Node . This is more relevant when I have Jenkins Cluster with Master and Slaves where I have Window Nodes and Linux Nodes ....
   
    - stages : Where the whole work happen . I have many diffent Stages in Pipeline . Inside Stages I have Stage , Inside Stage I have steps to execute that Stage
   
    <img width="600" alt="Screenshot 2025-03-19 at 11 26 16" src="https://github.com/user-attachments/assets/ca993f0d-d029-4fda-8cc8-169239466fe8" />

    - stage: I can have multiple Stage . Build Stage, Test Stage , Deploy Stage .....
   
  - Declarative checkout SCM : This come from my Coniguration where I define Git Repo, URL and the Branch (Pipeline Section).

  - There is 2 Main Advantage that comes with Scripted Pipelines :

    - First one : The moment I need kind of Conditional Logic or a little more Complex between Steps, I have very limited on UI Conigfuration, and I need multiple Plugin in different Jobs and I need to manage those Plugins (High Maintainence Cost) . So if I Script Pipeline it will be super easy to program write those conditional, I can easily express any kind of Conditional I can set Variable I can have Complex Logic without limited by UI
   
    - Second one : With Pipeline I don't have 7 different Build that are chained . Rather I have 1 Build with multiple Steps and each step is a Sub Job with its own configuration its own task but I don't have any effort in chaining them or connecting them bcs they are part of one same pipeline, one job and I have the seamless intergration of multiple steps . So Pipeline is bacsically the Superset of multiple freestyle jobs but in the much more simplytic ways
   
  <img width="600" alt="Screenshot 2025-03-19 at 11 49 23" src="https://github.com/user-attachments/assets/fb20cc90-eaf0-46c7-8ad0-5c1aa9d7154f" />

### Jenkins Syntax 

**post attr**

  - Execute some logic after All Stages executed

  - Conditions I can use: 

    - `always {}`: Whatever is in there it will execute no matter what . Alway Execute . 
   
    - `success {}` : I will execute a script here that is only relevant if the build succeeded
   
    - `failure {}` :  I will execute a script here that is only relevant if the build failed
   
  - To generalize in the post block, I can define expression of either build status or build status change

**Define Conditionals for each Stage**

  - For example : I only want to run the test in Development Branch build , I don't want to run build for other features Branch build or any other build .

  <img width="600" alt="Screenshot 2025-03-19 at 12 41 05" src="https://github.com/user-attachments/assets/41a29974-a7e0-466e-8da9-ae8be31b1a41" />

  - I can define : When Expressions

    ```
    when {
      expression {
        BRANCH_NAME = 'dev' 
      }
    }

    This Stage will execute if the current Branch is dev
    ```

  - The Branch name alway available through Jenkins ENV . For example : BRANCH_NAME = 'dev' .

  - I can also define OR like this `BRANCH_NAME = 'dev' || BRANCH_NAME = 'master'`

  - I can also define only build Application if the Code changes made in project : `BRANCH_NAME = 'dev' && CODE_CHANGES == true`  . The `CODE_CHANGES == true` is a Variable  defined on the global script


**Environment Variable**

  - Variables are available from Jenkins : `<Jenkins-enpoint>/env-vars.htlm`

    <img width="600" alt="Screenshot 2025-03-19 at 12 59 34" src="https://github.com/user-attachments/assets/90531c49-5f01-4aa3-a32b-33fc339970dd" />

  - In addition I can provide my Own ENV in : Whatever ENV I defined in here will be available for the whole Pipeline . The way to use that ENV is `"${envName}"`
 
  - Another Practice I can use is Credentials . Example I have the Stage that deploy my Application and that Stage I need Credentials of Development Server to connect to it and to copy the Artifact :

    - Define Credentials in Jenkins UI . Once Credentials defined I can use it in Jenkinsfile . To use that I can define a ENV like this in the environment : `SERVER_CREDENTIALS = credentials('')`
   
    - credentials('<credentials-id>'): is a function that will bind the credentials to my ENV . This is a plugin name Credentials Binding Plugin

**Wrapper Syntax**

  <img width="600" alt="Screenshot 2025-03-19 at 13 07 10" src="https://github.com/user-attachments/assets/1485699c-f749-4a42-bf2e-7ab26c21d2dd" />

  - If I need credentials for only Specific Stage I use Wrapper Syntax : `withCredentials([]) {}`. This take as parameter an Object `[]`. And that Object will be `usernamePassword(credentials: '<crendital_id>', usernameVariable: USER, passwordVariable: PWD)`, this function that let me get username and password of that credentials individually and I can store my username and password as a Varibable then Inside the block I can use USER and PWD Variable

  - I have to have 2 Plugin in Jenkins : Credentials Plugin (Allow me to create credentials inside Jenkins UI) , Credentials Binding Plugin (Allow me to use though credentials in Jenkinsfile through ENV) 

**Attr for build tool**

  <img width="600" alt="Screenshot 2025-03-19 at 13 15 14" src="https://github.com/user-attachments/assets/749a018a-8ed9-43d9-9046-2e9dac227faa" />

  - `tools {}` provide me with build tools for my project

  - Bcs if run `mvn install` or `gradle build` or `npm package` I need those tools available in Jenkinsfile

  - 3 tools that Jenkins support : Maven, Gradle and JDK . If I need Yarn or NPM I need to do other way

  - The way to define tools in `tools {}` is using the name that Jenkins supported and next is name of the tool (have to pre-installed in Jenkins)

**Parameters**

  <img width="600" alt="Screenshot 2025-03-19 at 13 30 11" src="https://github.com/user-attachments/assets/222f13e7-f901-434a-ad33-1f9817859c05" />

  - `parameters {}`

  - Maybe I have some external Configuration I want to provide to my build to change some behavior . Example I could have a build that deploy my Application to a Staging Server and I want to select which version of application I want to deploy

  - Type of Parameters :

    - `string(name: 'VERSION', defaultValue: '', descriptions: '')`
   
    - `choice(name: 'VERSION', choices['1.0.0', '2.0.0', '3.0.0'], descriptions: '')` : Example I can have a choices of Version
   
    - `booleanParam(name: 'executeTests', defaultValue: true, descriptionL '')`: example I want to skip certain Stage
   
  - Once those Types defined I can use it in different Stages :

    ```
      when {
        expression {
          params.executeTests == True
        }
      }

      - If executeTest is True execute that stage . if not don't execute
    ```

  - So I can use it in Conditionals, or I can get its values direct in Steps `"${params.VERSION}"`

**Using External Groovy Script**

<img width="600" alt="Screenshot 2025-03-20 at 13 16 14" src="https://github.com/user-attachments/assets/f05118ff-985a-49fd-a4f7-42cebd9a8dfd" />

  - Imagine scenario when I have all these Pipelines Steps, where I build front end and Run test , build backend and run test etc .... many different stages , many different logics . So good practice is to clean up Jenkinsfile and put those logis or script into its own files

  - The way to do that is to extract separate Groovy Script and extract it into external file

  - In the Pipeline of Jenkins file I can write Groovy script in `script {}`

  - All the ENV in Jenkinsfile are available in Groovy Script

**Input parameters in Jenkinsfile**

<img width="600" alt="Screenshot 2025-03-20 at 13 35 01" src="https://github.com/user-attachments/assets/363ca71a-6fba-4a04-b943-398c0e9ce914" />

  - I can implement in Jenkinsfile is to allow users to input some data on one of those execution steps .

  - For example : I am building Application I have run test I have build Artifact and in the end I want User to select which ENV should be deploy on Or maybe I am building a new Pipeline and I want user to Input a Version of the Artifact that should be deploy to a Staging Server

  - The way to do that is in : `input {}` inside -> I have attr `message "Select the ENV to deploy to"` | Also I have attr `ok "ENV selected"` . Now I have Option attr `parameters {}` this parameters attr is the same one as the Parameters above

  - When I run this build . This build will be pause until User select Value

### Multiple Branch Pipeline 

  - In Git repo usally I have 1 Main Branch and multiple children Branch like : Feature, Bugfix or etc ... . I can run test on all of them but I don't want to deploy them . However in Main Branch whenever a children Branch are merged Once it completed I want to Test, Build and Deploy afterward

  - There is 2 main point :

    - First, I need Pipeline for all the Branches so I can run test . The advantage of this is if I broke tests or something isn't working in the feature branch or bugfix branch before merging into Main Branch  
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

