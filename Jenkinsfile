pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_BACKEND = 'your-dockerhub-username/chat-app-backend:latest'
        IMAGE_FRONTEND = 'your-dockerhub-username/chat-app-frontend:latest'
        K8S_NAMESPACE = 'chat-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/AvishkarLakade3119/Chatbot-React-App.git', branch: 'main'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes..."

                    // Replace image names in manifests (if needed)
                    sh """
                    sed -i 's|image: .*backend.*|image: ${IMAGE_BACKEND}|' k8s/backend-deployment.yaml
                    sed -i 's|image: .*frontend.*|image: ${IMAGE_FRONTEND}|' k8s/frontend-deployment.yaml
                    """

                    // Apply all K8s manifests
                    sh """
                    kubectl apply -n ${K8S_NAMESPACE} -f k8s/
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "kubectl get all -n ${K8S_NAMESPACE}"
            }
        }
    }

    post {
        failure {
            echo "Deployment failed!"
        }
        success {
            echo "Deployment successful!"
        }
    }
}
