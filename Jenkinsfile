pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "chivy14082000/netccicd"
        DOCKER_REGISTRY_CREDENTIALS_ID = 'docker-hub'
        CONTAINER_NAME = "myapp_container" 
        OLD_IMAGE_TAG = "${DOCKER_IMAGE}:old"
        APP_ENV = "Production"
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

        stage('Update Configuration') {
            steps {
                script {
                    echo "Updating appsettings.json"

                    // Đường dẫn đến file appsettings.json
                    def appSettingsFile = "CICDNetcore/appsettings.json"
                    // Trường cần cập nhật
                    def fieldToUpdate = "Logging.LogLevel.Default"
                    // Giá trị mới bạn muốn gán
                    def newValue = "Information"

                    // Sử dụng PowerShell để đọc và cập nhật file JSON
                    powershell """
                        # Đọc nội dung file JSON
                        \$json = Get-Content -Path '${appSettingsFile}' | ConvertFrom-Json

                        # Cập nhật trường cụ thể
                        \$json.Logging.LogLevel.Default = '$newValue'

                        # Ghi lại file JSON
                        \$json | ConvertTo-Json -Depth 10 | Set-Content -Path '${appSettingsFile}'
                    """
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
                     // Kiểm tra nếu container đã tồn tại
                    def containerExists = powershell(script: "docker ps -aq -f name=${CONTAINER_NAME}", returnStdout: true).trim()

                   if (containerExists) {
                        // Kiểm tra xem container có đang chạy không
                        def containerRunning = powershell(script: "docker ps -q -f name=${CONTAINER_NAME}", returnStdout: true).trim()

                        if (containerRunning) {
                            echo "Stopping and removing the old running container"
                            // Dừng và xóa container đang chạy
                            powershell "docker stop ${CONTAINER_NAME}"
                        }

                        echo "Removing the old container"
                        // Xóa container cũ (đang dừng hoặc đã dừng)
                        powershell "docker rm ${CONTAINER_NAME}"
                    }

                    echo "Running the application from Docker image"
                    // Chạy container mới
                    powershell "docker run -d --name ${CONTAINER_NAME} -p 8081:80 ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Cleanup Old Image') {
            steps {
                script {
                    echo "Checking if the old image exists"
                    // Kiểm tra nếu image cũ tồn tại
                    def oldImageExists = powershell(script: "docker images -q ${DOCKER_IMAGE}:old", returnStdout: true).trim()

                    if (oldImageExists) {
                        echo "Removing old image"
                        // Xóa image cũ
                        powershell "docker rmi ${DOCKER_IMAGE}:old"
                    } else {
                        echo "No old image found"
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