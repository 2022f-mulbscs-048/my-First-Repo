pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/2022f-mulbscs-048/my-First-Repo.git'
        COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository...'
                git url: "${REPO_URL}"
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                sh 'docker-compose build'
            }
        }

        stage('Start Services') {
            steps {
                echo 'Starting containers...'
                sh 'docker-compose up -d'
            }
        }

        stage('Check Running Containers') {
            steps {
                echo 'Listing running containers...'
                sh 'docker ps'
            }
        }

        // Optional: Add test stage here if your app supports it

        stage('Stop Services') {
            steps {
                echo 'Stopping containers...'
                sh 'docker-compose down'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker resources...'
            sh 'docker system prune -f'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
