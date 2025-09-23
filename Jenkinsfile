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
        echo "Skipping SonarQube for now"
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
