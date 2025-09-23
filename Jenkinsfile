pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mynodeapp"
        CONTAINER_NAME = "mynodeapp_container"
        SONAR_TOKEN = credentials('SONAR_TOKEN') 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo "Installing dependencies..."
                bat 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Running Jest tests..."
                bat 'npm test'
            }
        }
        
        stage('Security') {
            steps {
                echo "Running npm audit for vulnerabilities..."
                bat 'npm audit --json > audit.json || exit 0'
            }
            post {
                always {
                    script {
                        def report = readJSON file: 'audit.json'
                        echo "Total vulnerabilities: ${report.metadata.vulnerabilities.total}"
                        echo "Critical: ${report.metadata.vulnerabilities.critical}, High: ${report.metadata.vulnerabilities.high}"
                        if (report.metadata.vulnerabilities.critical > 0) {
                            echo "⚠️ Critical issues found! Consider updating dependencies."
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Building and running Docker container..."
                bat '''
                    docker build -t %DOCKER_IMAGE% .
                    docker stop mynodeapp_container || echo "No container running"
                    docker rm mynodeapp_container || echo "No container to remove"
                    docker run -d -p 3000:3000 --name mynodeapp_container %DOCKER_IMAGE%
                '''
            }
        }
        stage('Release') {
            steps {
                echo "Tagging Docker image locally as release..."
                bat """
                    docker tag %DOCKER_IMAGE% %DOCKER_IMAGE%:release
                    docker images | findstr %DOCKER_IMAGE%
                """
            }
        }

        stage('Monitoring') {
    steps {
        echo "Checking container status and app health..."
        
        // Check if container is running
        bat "docker ps -f name=%CONTAINER_NAME%"

        // Check if app responds on port 3000
        powershell '''
            try {
                $response = Invoke-WebRequest -Uri http://localhost:3000 -UseBasicParsing
                if ($response.StatusCode -eq 200) {
                    Write-Host "✅ App is running and responding"
                } else {
                    Write-Host "⚠️ App returned status code $($response.StatusCode)"
                }
            } catch {
                Write-Host "❌ App not responding"
            }
        '''
    }
}

        
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
