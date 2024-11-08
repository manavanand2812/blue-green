pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'your-docker-image'  // Replace with actual image name
        DOCKER_REGISTRY = 'your-docker-registry'  // Replace with actual registry
        BLUE_SERVER = 'blue.example.com'  // Replace with actual server
        GREEN_SERVER = 'green.example.com'  // Replace with actual server
    }
    stages {
        stage('Clone Repo') {
            steps {
                script {
                    // Ensure correct branch and repo URL
                    git url: 'https://github.com/manavanand2812/blue-green.git', branch: 'main'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Tag and push Docker image to the registry
                    sh 'docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'
                    sh 'docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'
                }
            }
        }

        stage('Deploy to Blue') {
            steps {
                script {
                    // Pull and deploy Docker image to Blue server
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
                    // Restart Blue server services and shut down Green server
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml restart'"
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml down'"
                }
            }
        }

        stage('Deploy to Green') {
            steps {
                script {
                    // Pull and deploy Docker image to Green server
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
                    // Restart Green server services and shut down Blue server
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml restart'"
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml down'"
                }
            }
        }
    }
}
