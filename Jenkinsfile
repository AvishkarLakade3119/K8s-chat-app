pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        FRONTEND_IMAGE = 'avishkarlakade/chatapp-frontend'
        BACKEND_IMAGE = 'avishkarlakade/chatapp-backend'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/AvishkarLakade3119/K8s-chat-app.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    script {
                        docker.build("${BACKEND_IMAGE}:latest", '.')
                    }
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    script {
                        docker.build("${FRONTEND_IMAGE}:latest", '.')
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${BACKEND_IMAGE}:latest").push()
                        docker.image("${FRONTEND_IMAGE}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('k8s') {
                    sh 'kubectl apply -f backend-deployment.yaml'
                    sh 'kubectl apply -f frontend-deployment.yaml'
                }
            }
        }
    }
}
