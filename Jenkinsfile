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
                    echo "Running the application from Docker image"
                    powershell "docker run -d --name ${CONTAINER_NAME} -p 8000:80 ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
    }
}