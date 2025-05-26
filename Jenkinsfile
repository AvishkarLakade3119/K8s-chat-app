pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'  // Jenkins credentials ID for DockerHub
        DOCKERHUB_NAMESPACE = 'avishkarlakade'           // Your DockerHub username
        BACKEND_IMAGE = "avishkarlakade/chatapp-backend" // Correct string syntax, no ${} needed here
        FRONTEND_IMAGE = "avishkarlakade/chatapp-frontend"
        KUBECONFIG = '/home/jenkins/.kube/config'        // Adjust if needed in your Jenkins container
        K8S_NAMESPACE = 'chat-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AvishkarLakade3119/K8s-chat-app'
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    docker.build("${BACKEND_IMAGE}:latest", './backend')
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    docker.build("${FRONTEND_IMAGE}:latest", './frontend')
                }
            }
        }

        stage('Push Images') {
            steps {
                withDockerRegistry([credentialsId: "${DOCKERHUB_CREDENTIALS}", url: '']) {
                    docker.image("${BACKEND_IMAGE}:latest").push()
                    docker.image("${FRONTEND_IMAGE}:latest").push()
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                      kubectl -n ${K8S_NAMESPACE} set image deployment/backend-deployment chatapp-backend=${BACKEND_IMAGE}:latest
                      kubectl -n ${K8S_NAMESPACE} set image deployment/frontend-deployment chatapp-frontend=${FRONTEND_IMAGE}:latest

                      kubectl -n ${K8S_NAMESPACE} rollout status deployment/backend-deployment
                      kubectl -n ${K8S_NAMESPACE} rollout status deployment/frontend-deployment
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
