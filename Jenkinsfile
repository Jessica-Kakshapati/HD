pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mynodeapp"
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
        stage('Code Quality') {
            steps {
                 echo "Running SonarQube analysis..."
                 withSonarQubeEnv('SonarQubeServer') {
                    bat "sonar-scanner -Dsonar.projectKey=mynodeapp -Dsonar.sources=. -Dsonar.host.url=%SONAR_HOST_URL% -Dsonar.login=%SONAR_TOKEN%"
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

    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
