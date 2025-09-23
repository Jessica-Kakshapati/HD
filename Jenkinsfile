pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "hd"
        CONTAINER_NAME = "hd_container"
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
                            echo "Critical issues found! Consider updating dependencies."
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Starting services with Docker Compose..."
                bat '''
                    docker-compose down || echo "No existing services"
                    docker-compose build
                    docker-compose up -d
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

                // Wait for the app to respond on port 3001 (matches docker-compose)
                powershell '''
                    $maxRetries = 10
                    $delay = 5
                    $success = $false

                    for ($i=0; $i -lt $maxRetries; $i++) {
                        try {
                            $response = Invoke-WebRequest -Uri http://localhost:3001 -UseBasicParsing -TimeoutSec 3
                            if ($response.StatusCode -eq 200) {
                                Write-Host "✅ App is running and responding"
                                $success = $true
                                break
                            }
                        } catch {
                            Write-Host "App not responding yet. Retrying in $delay seconds..."
                        }
                        Start-Sleep -Seconds $delay
                    }

                    if (-not $success) {
                        Write-Host "❌ App did not respond after $($maxRetries * $delay) seconds"
                        exit 1
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
