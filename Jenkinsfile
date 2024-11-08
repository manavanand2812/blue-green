pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'your-docker-image'
        DOCKER_REGISTRY = 'docker.io'  // or your private registry URL
        DOCKER_USERNAME = credentials('bg')  // Credentials ID for Docker username
        DOCKER_PASSWORD = credentials('bg')  // Credentials ID for Docker password
        BLUE_SERVER = 'blue.example.com'
        GREEN_SERVER = 'green.example.com'
    }
    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/manavanand2812/blue-green.git'
            }
        }
        stage('Login to Docker Registry') {
            steps {
                script {
                    echo "Logging in to Docker registry"
                    echo "DOCKER_USERNAME: $DOCKER_USERNAME"  // Debugging step
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image"
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to registry"
                    sh 'docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'
                    sh 'docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'
                }
            }
        }
        stage('Deploy to Blue') {
            steps {
                script {
                    echo "Deploying to Blue server"
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker pull ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'"
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml up -d'"
                }
            }
        }
        stage('Verify Blue') {
            steps {
                input message: 'Verify if Blue deployment works', ok: 'Proceed'
            }
        }
        stage('Switch Traffic to Blue') {
            steps {
                script {
                    echo "Switching traffic to Blue server"
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml restart'"
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml down'"
                }
            }
        }
        stage('Deploy to Green') {
            steps {
                script {
                    echo "Deploying to Green server"
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker pull ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'"
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml up -d'"
                }
            }
        }
        stage('Verify Green') {
            steps {
                input message: 'Verify if Green deployment works', ok: 'Proceed'
            }
        }
        stage('Switch Traffic to Green') {
            steps {
                script {
                    echo "Switching traffic to Green server"
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml restart'"
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml down'"
                }
            }
        }
    }
}
