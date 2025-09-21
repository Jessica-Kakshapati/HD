pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "jessicak/myapp:%BUILD_NUMBER%"
    }

    stages {

        stage('Build') {
            steps {
                echo 'Installing dependencies and building the app...'
                bat 'npm install'
                bat 'npm run build'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                bat 'npm test'
                junit '**\\test-results.xml' // if you generate JUnit-compatible reports
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Running ESLint for code quality...'
                // Make sure ESLint is installed (either globally or in package.json devDependencies)
                bat 'npx eslint . || exit 0'
            }
        }

        stage('Security') {
            steps {
                echo 'Running npm audit for security vulnerabilities...'
                bat 'npm audit --json > audit-report.json || exit 0'
                // Optionally parse JSON and fail build if high severity issues found
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to staging environment with Docker...'
                // Stop old container if running
                bat 'docker stop myapp-staging || exit 0'
                bat 'docker rm myapp-staging || exit 0'
                // Build and run new container
                bat "docker build -t %DOCKER_IMAGE% ."
                bat "docker run -d -p 8080:8080 --name myapp-staging %DOCKER_IMAGE%"
            }
        }

        stage('Release') {
            steps {
                input message: 'Approve release to production?'
                echo 'Pushing Docker image to registry...'
                bat "docker tag %DOCKER_IMAGE% jessicak/myapp:latest"
                bat "docker push jessicak/myapp:latest"
            }
        }

        stage('Monitoring') {
            steps {
                echo 'Checking application health...'
                bat 'powershell -Command "try { Invoke-WebRequest -Uri http://localhost:8080/health -UseBasicParsing -ErrorAction Stop } catch { Write-Host \'Alert: App is down!\' }"'
            }
        }
    }

    post {
        failure {
            mail to: 'jessikakshapati@gmail.com',
                 subject: "Build failed in Jenkins: ${currentBuild.fullDisplayName}",
                 body: "Check Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
