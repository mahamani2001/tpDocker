pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *') // Corrected the cron expression format
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME_SERVER = 'maha487/mern-server' 
        IMAGE_NAME_CLIENT = 'maha487/mern-client' 
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'git@github.com/project.git', 
                    credentialsId: '760ee44f-d791-48d5-8ead-596e5b86ae14'
            }
        }
        stage('Build Server Image') {
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
                    }
                }
            }
        }
        stage('Build Client Image') {
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}")
                    }
                }
            }
        }
        stage('Scan Server Image') {
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy:latest image --exit-code 0 \
                    --severity LOW,MEDIUM,HIGH,CRITICAL \
                    ${IMAGE_NAME_SERVER}
                    """
                }
            }
        }
        stage('Scan Client Image') {
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy:latest image --exit-code 0 \
                    --severity LOW,MEDIUM,HIGH,CRITICAL \
                    ${IMAGE_NAME_CLIENT}
                    """
                }
            }
        }
        stage('Push Images to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
                        dockerImageServer.push()
                        dockerImageClient.push()
                    }
                }
            }
        }
    }
}
