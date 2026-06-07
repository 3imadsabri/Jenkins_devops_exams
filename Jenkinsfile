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

    }
}
