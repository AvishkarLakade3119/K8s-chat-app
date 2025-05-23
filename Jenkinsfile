pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-creds') // Jenkins ID created in Jenkins > Credentials
        FRONTEND_IMAGE = 'avishkarlakade/chatapp-frontend'
        BACKEND_IMAGE  = 'avishkarlakade/chatapp-backend'
        KUBE_NAMESPACE = 'chat-app'  // adjust if needed
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/AvishkarLakade3119/Chatbot-React-App.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $FRONTEND_IMAGE:latest ./frontend'
                sh 'docker build -t $BACKEND_IMAGE:latest ./backend'
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $FRONTEND_IMAGE:latest
                        docker push $BACKEND_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
