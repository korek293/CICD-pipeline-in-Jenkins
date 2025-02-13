@Library('my-shared-library') _
pipeline {
    agent any
    environment {
        BRANCH_NAME = "${env.GIT_BRANCH}"
        PORT = ''
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
		stage('Lint Dockerfile') {
            steps {
                script {
					dockerLint()
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
        stage('Build image') {
            steps {
                script {
                    if (BRANCH_NAME == "main") {
                        IMAGE_TAG = "toxic5/abahuslauski_application:nodemain-v1.0"
                        PORT = "3000"
                    } else if (BRANCH_NAME == "dev") {
                        IMAGE_TAG = "toxic5/abahuslauski_application:nodedev-v1.0"
                        PORT = "3001"
                    }
                    sh "docker build -t ${env.IMAGE_TAG} ."
                }
            }
        }
		stage('Trivy scanning') {
            steps {
                script {
                    def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${env.IMAGE_TAG}", returnStdout: true).trim()
					echo "Vulnerability Report:\n${vulnerabilities}"
                }
            }
        }
		stage('Push') {
            steps {
			  withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
												passwordVariable: 'dockerHubPassword', 
												usernameVariable: 'dockerHubUser')]) {
				sh 'echo "$dockerHubPassword" | docker login -u "$dockerHubUser" --password-stdin'
				sh "docker push ${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh "docker run -d -p ${PORT}:3000 ${IMAGE_TAG}"
                }
            }
        }
		stage('Run job') {
            steps {
                script {
                    if (BRANCH_NAME == "main") {
                        build job: 'Deploy_to_main'
                    } else if (BRANCH_NAME == "dev") {
                        build job: 'Deploy_to_dev'
                    }
                }
            }
		}
    }
}
