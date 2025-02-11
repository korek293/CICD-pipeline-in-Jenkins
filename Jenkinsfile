pipeline {
    agent any
    environment {
        BRANCH_NAME = "${env.GIT_BRANCH}" // Getting the branch name dynamically
        PORT = ''
        IMAGE_TAG = ''
        LOGO_PATH = ''
    }
    tools {
        nodejs 'node'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm  // Checkout the code from GitHub
            }
        }

        stage('Change Logo') {
            steps {
                script {
                    // Set correct logo path based on branch
                    if (BRANCH_NAME == "main") {
                        LOGO_PATH = "logo-main.svg"
                    } else if (BRANCH_NAME == "dev") {
                        LOGO_PATH = "logo-dev.svg"
                    }

                    // Replace the logo in the application BEFORE building the Docker image
                    echo "Changing logo for ${BRANCH_NAME} branch"
                    sh "cp ${LOGO_PATH} ./src/logo.svg"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Running npm install"
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "Running npm test"
                    sh 'npm test'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Set image tag and port based on branch
                    if (BRANCH_NAME == "main") {
                        IMAGE_TAG = "nodemain:v1.0"
                        PORT = "3000"
                    } else if (BRANCH_NAME == "dev") {
                        IMAGE_TAG = "nodedev:v1.0"
                        PORT = "3001"
                    }

                    // Build Docker image with the correct logo
                    echo "Building Docker image for ${BRANCH_NAME} branch"
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }

        stage('Stop and Remove Old Containers') {
            steps {
                script {
                    echo "Stopping and removing old containers"
                    sh '''
                    docker ps -q --filter "publish=3000" --filter "publish=3001" | xargs --no-run-if-empty docker stop
					docker ps -q --filter "publish=3000" --filter "publish=3001" | xargs --no-run-if-empty docker rm
                    '''
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                script {
                    echo "Deploying Docker container for ${BRANCH_NAME}"
                    sh "docker run -d -p ${PORT}:3000 ${IMAGE_TAG}"
                    echo "Application is running at http://localhost:${PORT}"
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean workspace after each build
        }
    }
}
