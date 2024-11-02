pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "chivy14082000/netccicd"
        DOCKER_REGISTRY_CREDENTIALS_ID = 'docker-hub'
        CONTAINER_NAME = "myapp_container"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Pulling code from Git repository"
                    checkout scm
                }
            }
        }
         stage('Check .NET Version') {
            steps {
               script {
                    echo "Check version"
                    powershell 'dotnet --version'
                }
            }
        }

        stage('Build') {
            steps {
               script {
                    echo "Building the .NET application"
                    powershell 'dotnet build'
                }
            }
        }

        stage('Test') {
            steps {
               script {
                    echo "Running tests"
                    powershell 'dotnet test'
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    echo "Publishing the application"
                    powershell 'dotnet publish -c Release -o publish'
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image"
                    powershell "docker build -t ${DOCKER_IMAGE}:latest ./CICDNetcore"
                }
            }
        }

        stage('Packaging/Pushing imagae') {

            steps {
                withDockerRegistry(credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/') {
                    powershell "docker push chivy14082000/netccicd"
                }
            }
        }

        stage('Run Application') {
            steps {
                script {
                    echo "Checking if the container is running"
                    def containerRunning = sh(script: "docker ps -q -f name=${CONTAINER_NAME}", returnStatus: true) == 0

                    if (containerRunning) {
                        echo "Stopping and removing the old container"
                        powershell "docker stop ${CONTAINER_NAME}"
                        powershell "docker rm ${CONTAINER_NAME}"
                    }

                    echo "Running the application from Docker image"
                    powershell "docker run -d --name ${CONTAINER_NAME} -p 8081:80 ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Cleanup Old Image') {
            steps {
                script {
                    echo "Checking if the old image exists"
                    def oldImageExists = sh(script: "docker images -q ${DOCKER_IMAGE}:old", returnStatus: true) == 0

                    if (oldImageExists) {
                        echo "Removing old image"
                        powershell "docker rmi ${OLD_IMAGE_TAG}"
                    }
                }
            }
        }
    }
    
    post {
        success {
            script {
                echo "Renaming the current image to 'old' for future cleanup"
                powershell "docker tag ${DOCKER_IMAGE}:latest ${OLD_IMAGE_TAG}"
            }
        }
        failure {
            script {
                echo "Image run failed, keeping the old image"
            }
        }
    }
}