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

        stage('Build') {
            steps {
                script {
                    // Run npm install
                    echo "Running npm install"
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Run npm test
                    echo "Running npm test"
                    sh 'npm test'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Conditional logic based on the branch name
                    if (BRANCH_NAME == "main") {
                        IMAGE_TAG = "nodemain:v1.0"
                        LOGO_PATH = "logo-main.svg"
                        PORT = "3000"
                    } else if (BRANCH_NAME == "dev") {
                        IMAGE_TAG = "nodedev:v1.0"
                        LOGO_PATH = "logo-dev.svg"
                        PORT = "3001"
                    }

                    // Build Docker image with the specified tag
                    echo "Building Docker image for ${BRANCH_NAME} branch"
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }

        stage('Stop and Remove Old Containers') {
            steps {
                script {
                    // Stop and remove previously running containers to avoid conflicts
                    echo "Stopping and removing old containers"
                    sh '''
                    docker ps -q --filter "name=nodemain" | xargs --no-run-if-empty docker stop
                    docker ps -q --filter "name=nodedev" | xargs --no-run-if-empty docker stop
                    docker ps -q --filter "name=nodemain" | xargs --no-run-if-empty docker rm
                    docker ps -q --filter "name=nodedev" | xargs --no-run-if-empty docker rm
                    '''
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                script {
                    // Run the Docker container based on the specified port
                    echo "Deploying Docker container for ${BRANCH_NAME}"
                    sh "docker run -d --expose ${PORT} -p ${PORT}:3000 ${IMAGE_TAG}"
                    echo "Application is running at http://localhost:${PORT}"
                }
            }
        }

        stage('Change Logo') {
            steps {
                script {
                    // Replace the logo in the application
                    echo "Changing logo for ${BRANCH_NAME} branch"
                    sh "cp ${LOGO_PATH} ./src/logo.svg"
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

