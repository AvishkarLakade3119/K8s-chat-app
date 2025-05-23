pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')  // Jenkins credential ID
        DOCKERHUB_USER = 'avishkarlakade'
        IMAGE_FRONTEND = 'chatapp-frontend'
        IMAGE_BACKEND = 'chatapp-backend'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AvishkarLakade3119/K8s-chat-app.git'
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    script {
                        sh 'docker build -t $DOCKERHUB_USER/$IMAGE_FRONTEND:latest .'
                    }
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    script {
                        sh 'docker build -t $DOCKERHUB_USER/$IMAGE_BACKEND:latest .'
                    }
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                script {
                    sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push $DOCKERHUB_USER/$IMAGE_FRONTEND:latest
                    docker push $DOCKERHUB_USER/$IMAGE_BACKEND:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes (Optional)') {
            steps {
                dir('k8s') {
                    sh 'kubectl apply -f .'
                }
            }
        }
    }
}
