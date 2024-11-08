pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'your-docker-image'
        DOCKER_REGISTRY = 'your-docker-registry'
        BLUE_SERVER = 'blue.example.com'
        GREEN_SERVER = 'green.example.com'
    }
    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/manavanand2812/blue-green.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    sh 'docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'
                    sh 'docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}'
                }
            }
        }
        stage('Deploy to Blue') {
            steps {
                script {
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
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml restart'"
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml down'"
                }
            }
        }
        stage('Deploy to Green') {
            steps {
                script {
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
                    sh "ssh ubuntu@${GREEN_SERVER} 'docker-compose -f docker-compose.green.yml restart'"
                    sh "ssh ubuntu@${BLUE_SERVER} 'docker-compose -f docker-compose.blue.yml down'"
                }
            }
        }
    }
}
