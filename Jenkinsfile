pipeline {
    agent any
    options {
        timestamps()
    }

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'     // Jenkins credentials ID
        DOCKERHUB_NAMESPACE = 'avishkarlakade'              // Your DockerHub username
        BACKEND_IMAGE = "avishkarlakade/chatapp-backend"
        FRONTEND_IMAGE = "avishkarlakade/chatapp-frontend"
        KUBECONFIG = '/home/jenkins/.kube/config'           // Ensure this exists inside the container
        K8S_NAMESPACE = 'chat-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Cloning repository...'
                git branch: 'main', url: 'https://github.com/AvishkarLakade3119/K8s-chat-app'
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Backend') {
                    steps {
                        echo 'Building backend Docker image...'
                        script {
                            docker.build("${BACKEND_IMAGE}:latest", './backend')
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        echo 'Building frontend Docker image...'
                        script {
                            docker.build("${FRONTEND_IMAGE}:latest", './frontend')
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                echo 'Pushing images to DockerHub...'
                script {
                    docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
                        docker.image("${BACKEND_IMAGE}:latest").push()
                        docker.image("${FRONTEND_IMAGE}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes cluster...'
                script {
                    sh """
                        kubectl -n ${K8S_NAMESPACE} set image deployment/backend-deployment backend=${BACKEND_IMAGE}:latest
                        kubectl -n ${K8S_NAMESPACE} set image deployment/frontend-deployment frontend=${FRONTEND_IMAGE}:latest

                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/backend-deployment
                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/frontend-deployment
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}
