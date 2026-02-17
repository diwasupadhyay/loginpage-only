pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'loginpage-app'
        DOCKER_TAG   = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
                bat "docker build -t %DOCKER_IMAGE%:latest ."
            }
        }

        stage('Deploy') {
            steps {
                bat 'docker-compose down || exit 0'
                bat 'docker-compose up -d --build'
            }
        }

        stage('Health Check') {
            steps {
                bat '''
                    echo Waiting for app to start...
                    timeout /t 10 /nobreak
                    curl -f http://localhost:3000/health || exit 1
                    echo App is healthy!
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
            bat 'docker-compose logs app || exit 0'
        }
        always {
            cleanWs()
        }
    }
}
