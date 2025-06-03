pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        DOCKERHUB_NAMESPACE = 'avishkarlakade'
        BACKEND_IMAGE = "avishkarlakade/chatapp-backend"
        FRONTEND_IMAGE = "avishkarlakade/chatapp-frontend"
        KUBECONFIG = '/home/jenkins/.kube/config'
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
                        # Create namespace if it does not exist
                        kubectl get namespace ${K8S_NAMESPACE} || kubectl create namespace ${K8S_NAMESPACE}

                        # Apply Kubernetes manifests
                        kubectl apply -n ${K8S_NAMESPACE} -f ./k8s/

                        # Expose services (in case they aren't already)
                        kubectl expose deployment frontend-deployment --type=NodePort --name=frontend --port=80 --target-port=80 -n ${K8S_NAMESPACE} || true
                        kubectl expose deployment backend-deployment --type=NodePort --name=backend --port=5001 --target-port=5001 -n ${K8S_NAMESPACE} || true
                        kubectl expose deployment mongodb-deployment --type=NodePort --name=mongodb --port=27017 --target-port=27017 -n ${K8S_NAMESPACE} || true

                        # Rollout status checks
                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/backend-deployment &
                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/frontend-deployment &

                        wait
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
