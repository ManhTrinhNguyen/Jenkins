pipeline {   
    agent any

    stages {

        stage("test") {
            steps {
               script {
                  echo "Testing Applications"
                  echo "Executing pipline for Branch ${BRANCH_NAME}"
               }
            }
        }
    
        stage("build") {
            when {
                expression {
                    BRANCH_NAME = "main"
                }
            }
            steps {
                script {
                    echo "Building Application"
                }
            }
        }

       
        stage("deploy"){
            when {
                expression {
                    BRANCH_NAME = "main"
                }
            }
            steps {
               echo "Deploying Application"
            }
        }
    } 
}
