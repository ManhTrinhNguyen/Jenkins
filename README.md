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

**There is 2 main point**

  - First, I need Pipeline for all the Branches so I can run test . The advantage of this is if I broke tests or something isn't working in the feature branch or bugfix branch before merging into Main Branch
   
  - Second, I need different behavior base on the Branch in Jenkinsfile . For example : If it Main Branch I want to build and deploy, if it not I want just Test and skip the rest
   
**How to go for those 2 Points** 

  - First : I need to define multiple branch pipelines . I need a pipelines for every branch Main, Feature, Bugfix etc ... I also want dynamically add them (I don't need to configuring the Branch Name) .

  - Second : Usally in projects I would have 1 Jenkinsfile that all branches share . Jenkinsfile in Main Branch would be the same as in all other Branches . So inside that Share Jenkinsfile, I need logic to define to build the Branch and to deploy it for Master branch and just execute the Test for all the other Branches 

**Set up in UI First Point**

  - Go for the Multiple Branch Pipeline

    - In Branch Sources : I can configure Git Repo for my Branch Pipeline -> Add Credentials -> Add Discover Branches (automatically added) -> Add Filter by name (regular expression)
   
      - In Filter branch by name with Regular Expression : `* (this will match all the branches)` .
     
  - The rest of configuration should be define in JenkinsFile

  - After created, A Scan Taks in Jenkins is running will scan each branch for a Jenkinsfile . Whenever it find a Jenkins file in the branch It will build a branch . If doesn't find Jenkinsfile it will skip and ignore it .

  - In the background individual pipeline, jobs get created for each branch that I have configured in regular expression and also have Jenkinsfile defined . Usually every branch has Jenkinsfile bcs I create new Branch from Main Branch

  - Technically, I could define multiple branches in a regular pipeline job, or define regular expression here as well . And it also build every Branch I define which is has Jenkinsfile . However I don't have all the Plugins and Variable that are useful for building multiple branches . Also I don't have a Nice transparent overview of which Branches are built


**Branch-based logic for Multiple Branch Pipeline | Second Point**

  - In Jenkinsfile, I will add logic that checks which Branch that currently building and base on the branch it will decide wheather to execute the Stage or not . 

    - I want the Test Stage to execute for all Branches
   
    - I want Build Stage and Deploy Stage to execute for only Main Branch . I can do that by using `when {expression {BRANCH_NAME = 'main'}}` . when expression syntax is like if conditions in programing language
   
      - `BRANCH_NAME` is a ENV that come out of the box in the multi branch pipeline jobs . Only available in this multi branch, Not in regular Pipeline
     
  - After that I will execute Scan MultiBranch Pipeline Now . This is like a Parent of all the sub pipelines for the individual branches . This will go through all the Branches match by regular expression and try to build all those branches with their respective changes

  - Now If I go back to Git repo and create a new branch . Then I go back to Jenkins and execute Scan MultiBranch Pipeline Now I will have that Branch 

### Credentials in Jenkins 

  - Crentials is a Plugin that I installed to Store and Manage Credentials Centrally

  - There is 3 Different Scope : Global , System Scope, Multi Branch Pipeline 

    - System Scope : Only available on Jenkins Server . For example, If I am a Jenkins Admin that administor a Jenkins server like Configuration Jenkins, Communication, Interface with other Services I will create a System Credentials . Bcs System Credentials are not visible or accessible by Jenkins Jobs .
   
    - Global : Accessible everywhere
   
    - Multi Branch Pipeline : This scope is limited to my Project and this come only with multi Branch Pipeline . And this acutally come from a Folder Plugin . Folder Plugin is basically organize my build jobs in folder
   
    - Note : This is a good way to organize Credentials but also separate them . If I have a very complex, big deployment project or 2 team that have their own project on the same Jenkins I can hide the credentials between project

  - Type of Credentials : Username and Password, Certificate, Secret File , Secret Text, Github

    - NOTE : Depending on other Plugin I have installed in Jenkins I might have new type of Credentials
   
---

## Project 2: Create a Jenkins Shared Library

**What is Jenkins Shared Library ?**

- Let's say I have a Java Maven Project that have multiple Microservices . Each Mirco have 1 Pipeline . I would build each Microservice as a Separate Application . Most of the build Logic are same like docker build, docker login etc ... logic . But I don't want to repeat myself with the same logic it is so unefficient 

- The efficient way to do is Jenkin Shared library . This is an extension to Pipeline and Jenkins Shared Library is its own Repository which is written in Groovy, and I can write all the Logic that is gonna be shared accross many different applications in the shared Library and reference that Logic from Jenkinfile in each project . This is for 1 project that have multiple Microservices

**Another use case**

<img width="600" alt="Screenshot 2025-03-26 at 12 20 33" src="https://github.com/user-attachments/assets/b4608619-ade7-4337-94b6-d2e78cb51f8e" />

- What about for a company have multiple Project . All the Projects may not use the same technology stack however some of the logic still the same like : Push Image to Nexus, ECR, ..., to send Notifications to the same Slack Channels . All of this are something that can be shared accross multiple projects in the same company in their pipelines . Instead if 1 team write this logic another team can use it

**Create the Shared Library**

- In order to use Share Library I have to create Repository , Wirte Groovy Code , make Share Library available global or for project, use Share Library in Jenkinfile

- Create new Groovy project in IntelliJ . Bcs Jenkin share library has written in Groovy 

**Structure of Jenkins Share Lib**

<img width="600" alt="Screenshot 2025-03-26 at 12 52 15" src="https://github.com/user-attachments/assets/5e743ee4-30fe-40fc-87e5-d7c6b3804026" />

- Main folder `vars` : This folder include all the function that I will execute or I can call from Jenkinfile . And each function will be its own Groovy file

- `src` Source Folder : Similar to Java src folder . This is where I can have any helper, logic, utility code for those function

- Resources Folder : Which allow me to use external Library for non Groovyfile. I can help SQL Script there, Power shell, shell scripts, JSON files, etc ...

**vars folder**

- The way this work in `vars` . All the files that I have here all those executable functions, I am referencing them by the name of the file . I will use that to call that function in Jenkinfiles

- For example : This is a  `buildJar.groovy` function

```
#!/user/bin/env groovy

# This line above is let my editor detect that I am working with Groovy script . I can use the same annotation in Jenkinfiles to let my editor know that I am working with Groovy script bcs the function has no .groovy

def call () {
  echo 'I am building a Jar'
  sh 'mvn package'
}
``` 

- For example : this is a `buildImage.groovy` function

```
#!/user/bin/env groovy

# This line above is let my editor detect that I am working with Groovy script . I can use the same annotation in Jenkinfiles to let my editor know that I am working with Groovy script bcs the function has no .groovy

def call () {
   echo "building the docker image..."
   withCredentials([usernamePassword(credentialsId: 'aws_ECR_credential', passwordVariable: 'PASS', usernameVariable: 'USER')]){
      sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
      sh "echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}"
      sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
}
```

- Once I have those 2 funcion (I can add more) I can push those two in the Repository

**Make Share Library Available**

- Go to Manage Jenkin -> System -> Global Pipeline Library -> Add Library -> Provide Git URL (Mordern SCM) , Add credentials 

- Default version : This can be either branch, this can also be commit hash or this can be a tag . Just like any other Application I should version my Jenkin Share Lib so whenever I make change I create a new version, tag and then I can reference that tag here . If I define Main Branch here then every new change or every commit to a Repo will be immediately available in whole Jenkin for all the Pipeline and it is a breaking change then my Jenkin file or my Pipeline will not work anymore so having a fixed version is a good idea

- With `Allow default version to be overridden` I can override that default version in my Jenkinfile If I want to

**User Share Library in Jenkinfile**

- In order to use function in Jenkins Share Lib I have to import that Share Lib : `@Library('jenkins-shared-lib')` . I can also choose specific version if I have `@Library('jenkins-shared-lib@2.0')`

  - 'jenkins_shared_lib' : is the name that I gave when defining in global library
 
  - !!! NOTE : If I don't have import statement or variable definition directly after the Import I have to add _ `@Library('jenkins-shared-lib')_` .
 
  ```
  stage("build jar"){
    steps {
      script {
        buildJar() # I call the function by the name of the file this will reference the function I defined in Share Lib
      }
    }
  }
  ```

**Using Parameters**

```
#!/user/bin/env groovy

# This line above is let my editor detect that I am working with Groovy script . I can use the same annotation in Jenkinfiles to let my editor know that I am working with Groovy script bcs the function has no .groovy

def call () {
   echo "building the docker image..."
   withCredentials([usernamePassword(credentialsId: 'aws_ECR_credential', passwordVariable: 'PASS', usernameVariable: 'USER')]){
      sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
      sh "echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}"
      sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
}
```

- DOCKER_REPO, IMAGE_NAME, DOCKER_REPO_SERVER is a Global variable . I already has it available so I don't need to access it as Parameter

- I need to pass a Parameter is when I don't have that Variable in Globol for example I don't have DOCKER_REPO, IMAGE_NAME, DOCKER_REPO_SERVER as ENV in global

```
#!/user/bin/env groovy

# This line above is let my editor detect that I am working with Groovy script . I can use the same annotation in Jenkinfiles to let my editor know that I am working with Groovy script bcs the function has no .groovy

def call (String DOCKER_REPO, String IMAGE_NAME, String DOCKER_REPO_SERVER) {
   echo "building the docker image..."
   withCredentials([usernamePassword(credentialsId: 'aws_ECR_credential', passwordVariable: 'PASS', usernameVariable: 'USER')]){
      sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
      sh "echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}"
      sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
}

```

- To pass the value to those Parameter above . In Jenkinfile I can pass a Parameter like this :

```
  stage("build Image"){
    steps {
      script {
        buildImage 'nguyenmanhtrinh/repo' 'java-maven' 'docker.io'
      }
    }
  }
```

**Extract Logic into Groovy Class**

- Let say I decide to have separate individual functions for each Docker Command, or maybe I have multiple maven command or git command that I want to execute separately as a separate function that would mean I have multiple groovy file, each of them implementing their own logic, and these command may share credentials, Image name, or Repository url etc and I may end up duplicating a lot of stuff in each function own implementation . In this case make sense to extract these logic and it infomation that these function share into a single file in groovy script and then reference it from here

- I can do it in `src` package . I can add Groovy script here that will hold all the logic centrally . Create package `com.example`

- In `src/com.example/Docker.groovy`

```
#!/user/bin/env groovy
package com.example

class Docker implements Serializable {

}
```

- Add Implement Serializable to the class to support saving the State of the execution if the pipeline is paused and resumed

- Another thing is this method are exposed Globally so when we use them in Jenkinfile we have all of those things like `withCredentials()` that are available in Jenkinfile that also available directly in Jenkin Shared Lib `vars/<all-the-functions>` but It's not available in `src/com.example.Docker.groovy`

- In order to make all the pipeline and syntax available in `src/com.example.Docker.groovy` I need to pass the Parameter those methods in `vars` folder and I will call it `script` . `script` bascially holds all the infomation including environments all the command and methods for the pipeline . `Script` will allow me to execute all those command . 

```
#!/user/bin/env groovy
package com.example

class Docker implements Serializable {

  def script

  Docker(script) {
    this.script = script
  }
}
```

- Create Single method call `buildDockerImage`

```
#!/user/bin/env groovy
package com.example

class Docker implements Serializable {

  def script

  Docker(script) {
    this.script = script
  }

  def buildDockerImage (String DOCKER_REPO, String IMAGE_NAME, String DOCKER_REPO_SERVER) {
   script.echo "building the docker image..."
   script.withCredentials([script.usernamePassword(credentialsId: 'aws_ECR_credential', passwordVariable: 'PASS', usernameVariable: 'USER')]){
      script.sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
      script.sh "echo '${script.PASS}' | docker login -u '${script.USER}' --password-stdin '${script.DOCKER_REPO_SERVER}'"
      script.sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
  }  
}

  - With these credentials like this  ${PASS} ${USER}, I need to use differnet syntax . This is to make sure that the $ that I use to hightlight a variable isn't mistaken for character in the username and password which can cause authentication error I will add single quite and script I put in bracket like this (It has to be in double quote) : '${script.PASS}'
```

- Now I need to call it from `vars/buildImage.groovy`

```
import com.example.Docker

def call(String DOCKER_REPO, String IMAGE_NAME, String DOCKER_REPO_SERVER){

  return new Docker(this).buildDockerImage(DOCKER_REPO, IMAGE_NAME, DOCKER_REPO_SERVER)
}

- I will pass the Object that has access to all the things that are available in Jenkinfile and I gonna do that using `this`, So I am passing the context from `vars/buildImage.groovy` to Docker Class in `src/com.example/Docker` .

- I still need call buildImage function in Jenkinsfile . . Nothing change in Jenkinfile

- NOTE : I could use acutally use classes and functions from the `src` folder I don't have to access them through global Variable . However best practice is to call function from `vars` folder
```

- I can Also Separate buildDockerImage in Docker Class into 3 Separte function like this : dockerBuildImage, dockerLogin, dockerPush . And then I will create that 3 function in `vars` folder 

<img width="600" alt="Screenshot 2025-03-26 at 15 08 09" src="https://github.com/user-attachments/assets/1423c092-5b2d-4851-b63b-42c79e9ffbfd" />

<img width="600" alt="Screenshot 2025-03-26 at 15 10 26" src="https://github.com/user-attachments/assets/71eea3c4-b057-4418-8dfa-3861e93e992b" />

<img width="600" alt="Screenshot 2025-03-26 at 15 11 25" src="https://github.com/user-attachments/assets/b9beffb9-62dd-47b7-883e-407e3daebfb6" />

**Project Scoped Share Library**

- Let say I have Jenkins share Lib but for only my project, other project or other team may not need them maybe I want to share them for my own pipeline . So I don't need to add Share Lib in Global Library and instead I will refererence library directly from Jenkinsfile

```
library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
     remote: https://github.com/ManhTrinhNguyen/Jenkins-Docker-Excercise-Shared-Library.git,
     credentialsId: 'github-credentials'
    ]
)
```
---

## Project 3: Configure Webhook to Trigger CI Pipeline Automatically on Every Change

- I want the trigger the build automatically whenever a commit is pushed into Git Repository 

- Other way to trigger build by Scheduling . For example I want this build run every 2 hours or every day at specific time

**For Gitlab**

- First I need Jenkin Plugin call : Gitlab

- Once the plugin install I should see the Gitlab Configuration in Manage -> System

- Config Gitlab in GitLab Configuration - Name, Host URL `gitlap.com` (not the project URL) -> Create API Token for Gitlab

- To Create Gitlab API Token in Gitlab -> Go to user Profile -> Choose Preferences -> Choose Access Token

- In my Pipeline Configuration -> I have Gitlab Connection -> Scroll down to Build Trigger I have Build when changed is pushed to Gitlab

- To configure Gitlab to automatically send notification to Jenkins  whenever commit happen or push happen -> In Gitlab Setting -> Choose Integration -> Scroll down and choose Jenkin -> Choose Push -> Put in Jenkins URL include the Port (If I run Jenkin on Localhost It will not work) -> Put project name (project name is a Job Name inside Jenkin) -> Put username and password of Jenkin

**For Github**

- Go to your GitHub repo → Settings → Webhooks

- Click "Add webhook"

- Payload URL: `http://<your-jenkins-domain>/github-webhook/`

- Content type: application/json

- Choose : Just the push event

- Save webhook

**For Multibranch pipeline for every Repository**

<img width="600" alt="Screenshot 2025-03-27 at 09 43 53" src="https://github.com/user-attachments/assets/41f70a4b-aa1d-49e0-ae0f-c2365357aec5" />

- I need another Plugin call Multibranch Scan Webhook Trigger

- Once Installed I have `Scan by Webhook` in Multi branch Configuration -> Choose Scan by Webhook I have Trigger Token (This is a token that I can name whatever I want this Token will be use for the communication between Gitlab and Jenkin, or Github and Jenkins ...)

- To use that Token I will go to Gitlab -> choose Webhook (Webhook is basically same Integration) . The way it work is I will tell Gitlab to send Jenkin a Notification on a Specific URL using that Token and when Jenkin receive a request it will check a Token and it will trigger a multibranch pipeline which has scan by webhook configured for that specific token . I don't need the Secret Token 

- `Periodically if not otherwise run` : is for shedule the run 

---

## Project 4: Dynamically Increment Application Version in Jenkins Pipeline

**Software Version**

<img width="923" alt="Screenshot 2025-03-27 at 10 43 11" src="https://github.com/user-attachments/assets/6c03da2c-3c98-429c-aed0-74a24b450ba7" />

- I decide how I version my Application . When I decide to update feature or fix bug of my App, I want to version that and release it

- 3 Parts of Version :

  - Major: Contain big change, bunch of new feature, breaking change ...
 
  - Minor Version: This is also when I have a lot of Changed but they are not as impactful as a Major version Upgrade
 
  - path or incremental version : Minor changes or bug fixe, doesn't change API
 
  - -SNAPSHOT is a suffix that I can add to my version . Suffix is for more infomation

**How to Increment the version ?**

- I want to be able to automatically increase that version inside my build . So When I commit changes to Jenkins, Jenkins build pipeline basically should increment the version and release a new application . This should all happen imediately .

- Each of those build tool they are all have commands or plugins that are used to increment the version .

**Increment version for Java Maven**

<img width="925" alt="Screenshot 2025-03-27 at 10 59 35" src="https://github.com/user-attachments/assets/4532c41c-bdc8-41ac-9202-56d3b622bbff" />

- Command : `mvn build-helper:parse-version versions:set -DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion} versions:commit`

  - `parse-version` : It goes and find pom file and it find a version tag . When it find a version tag it parses the version inside into 3 parts Major, Minor, Increment
 
  - `version:set` : Set a version between version tag
 
  - `-DnewVersion` : This is a Parameter for `versions:set`
 
  - `\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion}` : This is How I know what is the next version that I need to increment to. I use `next` so it the parse-version know that Incremental part need to increase . If I don't use `next` it will keep it as it is
 
  - `version:commit` : Replace pom.xml with new Version
 
- To execute the command in the sh command in Jenkinsfile the syntax to escape dollar sign a little different: `mvn build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} versions:commit`

- $BUILD_NUMBER : is a ENV that Jenkin make available for us and my pipeline job 

**For Docker Image Version** 

<img width="600" alt="Screenshot 2025-03-27 at 11 46 50" src="https://github.com/user-attachments/assets/a3388017-b0cb-47e1-aa6d-430c7da78a77" />

- I need to read pom.xml and access the version value then set it as a Variable

- To read pom.xml file and looking for version tag inside and put the `(.+)` regular expression to dynamic set a version value and set a variable to it called matcher: `def matcher = readFile(pom.xml) =~ <version>(.+)</version>` . This will give me an array of all the verions tags that it could find in this case I just have 1 and also the version value (child of version tag), so I would get that element like this : `matcher[0][1]`

**Dynamic increment App version in Nodejs for Jenkins**

<img width="600" alt="Screenshot 2025-03-27 at 11 30 10" src="https://github.com/user-attachments/assets/2c2feb41-ee25-4b68-8b2b-1c61aeb3faba" />

- npm install -D auto-version-js

 - npx auto-version --patch # +0.0.1
 - npx auto-version --minor # +0.1.0
 - npx auto-version --major # +1.0.0
 - npx auto-version # no args is equivalent to --patch
   
- Install Plugin to use "readJSON file" : Pipeline: Utility Steps 





















