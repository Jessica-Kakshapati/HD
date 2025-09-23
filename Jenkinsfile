pipeline {
    agent any


    environment {
        // Docker image name
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
                bat 'npm test'
            }
        }

        stage('Code Quality') {
            steps {
                echo "Downloading and running SonarQube analysis..."
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat '''
                        curl -sSLo sonar-scanner-cli.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006.zip
                        powershell -Command "Expand-Archive -Force sonar-scanner-cli.zip sonar-scanner-cli"
                        set PATH=%CD%\\sonar-scanner-cli\\sonar-scanner-5.0.1.3006\\bin;%PATH%
                        sonar-scanner
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Building and running Docker container..."
                bat '''
                    docker build -t %DOCKER_IMAGE% .
                    docker rm -f mynodeapp_container || exit 0
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
