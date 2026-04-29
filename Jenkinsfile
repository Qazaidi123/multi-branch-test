pipeline {
    agent any

    environment {
        
        IMAGE_NAME = "multi-branch-image"
        
    }

    stages {


        //  Run on ALL branches
        stage("Build Images") {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BRANCH_NAME}-${BUILD_NUMBER} ."
                
            }
        }
    

        // ONLY MAIN
        stage("Run Container") {
            when {
                branch 'main'
            }
            steps {
               sh "docker run -d --name multicon-${BRANCH_NAME}-${BUILD_NUMBER} -p 9091:80 ${IMAGE_NAME}:${BRANCH_NAME}-${BUILD_NUMBER}"
              }
            }
        }
    }

    
