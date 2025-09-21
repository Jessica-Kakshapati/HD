pipeline {
    agent any

    tools {
        // Example: Gradle or Maven for Java, or Python for pytest
        jdk 'jdk17'  // if Java project
    }

    stages {
        stage('Build') {
            steps {
                echo "Building the application..."
                bat 'gradlew.bat clean build'  // Java Gradle build (use mvn if Maven)
                // OR if using Docker:
                // bat 'docker build -t myapp:latest .'
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                bat 'gradlew.bat test'  // Java example
                // Python example: bat 'pytest tests/'
            }
        }

        stage('Code Quality') {
            steps {
                echo "Running SonarQube analysis..."
                // Requires SonarQube plugin + server config in Jenkins
                bat 'gradlew.bat sonarqube'
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to staging environment..."
                // Run Docker container
                bat 'docker run -d -p 8080:8080 myapp:latest'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
