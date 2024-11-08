pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'your-docker-image'
        DOCKER_REGISTRY = 'your-docker-registry'
        DOCKER_USERNAME = credentials('blue-green')  // Use Jenkins credentials ID 'blue-green' for Docker username
        DOCKER_PASSWORD = credentials('blue-green')  // Use Jenkins credentials ID 'blue-green' for Docker password
        BLUE_SERVER = 'blue.example.com'
        GREEN_SERVER = 'green.example.com'
    }
    stages {
        stage('Clone Repo') {
            steps {
                // Clone from the 'main' branch
                git branch: 'main', url: 'https://github.com/manavanand2812/blue-green.git'
            }
        }
        stage('Login to Docker Registry') {
            steps {
                script {
                    // Login to Docker Registry
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    // Tag and push the Docker image
                    sh 'docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'
                    sh 'docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'
                }
            }
        }
        stage('Deploy to Blue') {
            steps {
                script {
                    // Deploy to the Blue server
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
                    // Restart Blue deployment and bring down the Green deployment
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml restart'"
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml down'"
                }
            }
        }
        stage('Deploy to Green') {
            steps {
                script {
                    // Deploy to the Green server
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
                    // Restart Green deployment and bring down the Blue deployment
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml restart'"
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml down'"
                }
            }
        }
    }
}
