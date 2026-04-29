pipeline {
    agent any

    environment {
        SONAR_HOME = tool "Sonar"
        FRONTEND_IMAGE = "qazaidi123/frontend"
        BACKEND_IMAGE = "qazaidi123/backend"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage("Clone") {
            steps {
                git url: "https://github.com/Qazaidi123/kubernetes.git",
                    credentialsId: "git-creds"
            }
        }

        // ✅ Run on ALL branches (dev + PR + main)
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectKey=NETLIPROJ"
                }
            }
        }

        // ✅ Run on ALL branches
        stage("Build Images") {
            steps {
                sh "docker build -t $FRONTEND_IMAGE:$IMAGE_TAG ./frontend"
                sh "docker build -t $BACKEND_IMAGE:$IMAGE_TAG ./backend"
            }
        }

        // ✅ Run on ALL branches
        stage("Trivy Scan") {
            steps {
                sh "trivy image --severity CRITICAL --exit-code 0 $FRONTEND_IMAGE:$IMAGE_TAG"
                sh "trivy image --severity CRITICAL --exit-code 0 $BACKEND_IMAGE:$IMAGE_TAG"
            }
        }

        // 🔒 ONLY MAIN
        stage("Push to DockerHub") {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $FRONTEND_IMAGE:$IMAGE_TAG
                    docker push $BACKEND_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }

        // 🔒 ONLY MAIN
        stage("Deploy to EKS") {
            when {
                branch 'main'
            }
            steps {
                withAWS(credentials: 'AWS-CREDENTIALS') {
                    sh '''
                    aws eks --region ap-south-1 update-kubeconfig --name ekscluster
                    kubectl apply -f k8s/

                    kubectl set image deployment/frontend-deployment frontend-container=$FRONTEND_IMAGE:$IMAGE_TAG
                    kubectl set image deployment/backend-deployment backend-container=$BACKEND_IMAGE:$IMAGE_TAG

                    kubectl rollout status deployment/frontend-deployment
                    kubectl rollout status deployment/backend-deployment
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build Success"
        }
        failure {
            echo "Build Failed"
        }
    }
}
