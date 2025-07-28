pipeline {
    agent any
    environment {
        REPO_URL = 'https://github.com/2022f-mulbscs-048/my-First-Repo.git'
    }
    stages {
        stage('Checkout') {
            steps { git url: "${REPO_URL}" }
        }
        stage('Build Application') {
            steps {
                echo 'Building app WAR file...'
                // Example for Maven:
                sh 'cd Docker-files/app && mvn clean package -DskipTests'
                // or for Gradle:
                // sh 'cd Docker-files/app && ./gradlew clean build -x test'
            }
        }
        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images via docker-compose...'
                sh 'docker-compose build'
            }
        }
        stage('Start Services') {
            steps { sh 'docker-compose up -d' }
        }
        stage('Health Check / Smoke Test') {
            steps {
                echo 'Running a quick connectivity check...'
                sh 'curl -f http://localhost:9090/ || exit 1'
            }
        }
        stage('Teardown') {
            steps { sh 'docker-compose down' }
        }
    }
    post {
        always {
            echo 'Cleaning up Docker resources.'
            sh 'docker system prune -f'
        }
        success { echo 'Pipeline succeeded!' }
        failure { echo 'Pipeline failed.' }
    }
}
