pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "chivy14082000/netccicd"
        DOCKER_REGISTRY_CREDENTIALS_ID = 'docker-hub'
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
                    powershell "docker build -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

         stage('Push Docker Image') {
            steps {
                withCredentials(credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/') {
                    script {
                        echo "Pushing Docker image to registry"
                        powershell "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

    }
}