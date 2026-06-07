pipeline {
    agent any

    stages {

        stage('Verify Workspace') {
            steps {
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Docker Validation') {
            steps {
                sh 'docker compose config'
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                docker build -t imadsabri01/movie-service:latest ./movie-service
                docker build -t imadsabri01/cast-service:latest ./cast-service
                '''
            }
        }

    }
}
