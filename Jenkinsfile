pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rishirk408/cicd-demo-api"
    }

    stages {
        stage('Checkout') {
            steps {
                // gets code from GitHub
                checkout scm
            }
        }

        stage('Unit Tests') {
            steps {
                // Run tests with pip + pytest on Windows
                bat """
                  pip install -r requirements.txt
                  pytest
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                // Use Jenkins build number as image tag
                bat """
                  docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .
                  docker tag %DOCKER_IMAGE%:%BUILD_NUMBER% %DOCKER_IMAGE%:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_PASS'
                )]) {
                    bat """
                      docker login -u %DOCKERHUB_USER% -p %DOCKERHUB_PASS%
                      docker push %DOCKER_IMAGE%:%BUILD_NUMBER%
                      docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Update the running deployment in Docker Desktop Kubernetes
                bat """
                  kubectl set image deployment/cicd-demo-api api=%DOCKER_IMAGE%:%BUILD_NUMBER% --record
                  kubectl rollout status deployment/cicd-demo-api
                """
            }
        }
    }
}

