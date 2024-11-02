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
    }
}