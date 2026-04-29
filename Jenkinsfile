pipeline {
    agent any

    environment {
        SONAR_HOME = tool "Sonar"
        IMAGE_IMAGE = "qazaidi123/multibranch"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage("Clone") {
            steps {
                git url: "https://github.com/Qazaidi123/multi-branch-test.git",
                    credentialsId: "git-creds"
            }
        }


        //  Run on ALL branches
        stage("Build Images") {
            steps {
                sh "docker build -t $IMAGE_NAME:$IMAGE_TAG"
                
            }
        }
    

        // ONLY MAIN
        stage("Run Container) {
            when {
                branch 'main'
            }
            steps {
                docker run -d --name multicon -p 9091:80 ${IMAGE_NAME}:${IMAGE:TAG}
              }
            }
        }
    }

    
