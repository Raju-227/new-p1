pipeline {
    agent any

    environment {
        IMAGE_NAME = "yourdockerhubusername/my-food-webpage"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Docker Swarm') {
            steps {
                sshagent(['swarm-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no user@swarm-manager-host '
                        docker stack deploy -c /path/to/docker-compose.yml food-webpage-stack
                    '
                    """
                }
            }
        }
    }
}

