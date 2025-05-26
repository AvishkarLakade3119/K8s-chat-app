pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'  // Jenkins credentials ID for DockerHub
        DOCKERHUB_NAMESPACE = 'avishkarlakade' // Your DockerHub username
        BACKEND_IMAGE = "${avishkarlakade/chatapp-backend}/chatapp-backend"
        FRONTEND_IMAGE = "${avishkarlakade/chatapp-frontend}/chatapp-frontend"
        KUBECONFIG = '/home/jenkins/.kube/config'  // Adjust this path inside Jenkins container
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
                    // Update deployments to use the new images
                    sh """
                      kubectl --kubeconfig=${KUBECONFIG} -n ${K8S_NAMESPACE} set image deployment/backend-deployment backend-container=${BACKEND_IMAGE}:latest
                      kubectl --kubeconfig=${KUBECONFIG} -n ${K8S_NAMESPACE} set image deployment/frontend-deployment frontend-container=${FRONTEND_IMAGE}:latest

                      # Wait for rollout to finish
                      kubectl --kubeconfig=${KUBECONFIG} -n ${K8S_NAMESPACE} rollout status deployment/backend-deployment
                      kubectl --kubeconfig=${KUBECONFIG} -n ${K8S_NAMESPACE} rollout status deployment/frontend-deployment
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
