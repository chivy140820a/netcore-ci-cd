pipeline {
    agent any
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
                    sh 'dotnet --version'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building the .NET application"
                    sh 'dotnet build'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "Running tests"
                    sh 'dotnet test'
                }
            }
        }

    }
}