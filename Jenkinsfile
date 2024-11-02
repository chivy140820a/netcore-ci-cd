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
         stage('Check .NET Version 1') {
            steps {
                powershell '''
                    dotnet --version
                '''
            }
        }

        stage('Build') {
            steps {
                powershell '''
                    dotnet build
                '''
            }
        }

        stage('Test') {
            steps {
                powershell '''
                    dotnet test
                '''
            }
        }

    }
}