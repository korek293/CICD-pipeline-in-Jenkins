pipeline {
    agent any
    environment {
        BRANCH_NAME = "${env.GIT_BRANCH}"
        PORT = ''
        IMAGE_TAG = ''
        LOGO_PATH = ''
    }
    tools {
        nodejs 'node'
    }
    stages {
        stage('Logo') {
            steps {
                script {
                    if (BRANCH_NAME == "main") {
                        LOGO_PATH = "logo-main.svg"
                    } else if (BRANCH_NAME == "dev") {
                        LOGO_PATH = "logo-dev.svg"
                    }
                    sh "cp ${LOGO_PATH} ./src/logo.svg"
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    sh 'npm install'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    sh 'npm test'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    if (BRANCH_NAME == "main") {
                        IMAGE_TAG = "nodemain:v1.0"
                        PORT = "3000"
                    } else if (BRANCH_NAME == "dev") {
                        IMAGE_TAG = "nodedev:v1.0"
                        PORT = "3001"
                    }
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }

        stage('Stop and Remove Old Containers') {
            steps {
                script {
                    sh "docker ps -q --filter 'publish=3000' --filter 'publish=3001' | xargs --no-run-if-empty docker stop"
					sh "docker ps -q --filter 'publish=3000' --filter 'publish=3001' | xargs --no-run-if-empty docker rm"
                }
            }
        }
        stage('Deploy to Docker') {
            steps {
                script {
                    sh "docker run -d -p ${PORT}:3000 ${IMAGE_TAG}"
                    echo "Application is running on http://localhost:${PORT}"
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
