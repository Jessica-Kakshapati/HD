pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mynodeapp"
    }

    stages {
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

        stage('Code Quality') {
            steps {
                echo "Running SonarQube analysis..."
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat '''
                        curl -sSLo sonar-scanner-cli.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006.zip
                        powershell -Command "Expand-Archive -Force sonar-scanner-cli.zip sonar-scanner-cli"
                        sonar-scanner-cli\\sonar-scanner-5.0.1.3006\\bin\\sonar-scanner ^
                          -Dsonar.projectKey=mynodeapp ^
                          -Dsonar.sources=. ^
                          -Dsonar.host.url=http://your-sonarqube-server:9000 ^
                          -Dsonar.login=%SONAR_TOKEN%
                    '''
                }
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

        stage('Monitoring') {
            steps {
                echo "Checking application health..."
                bat '''
                    curl -f http://localhost:3000 || echo "App is down!"
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
