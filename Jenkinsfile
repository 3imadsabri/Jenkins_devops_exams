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
	stage('Push Images') {
	    steps {
	        withCredentials([usernamePassword(
	            credentialsId: 'dockerhub-creds',
	            usernameVariable: 'DOCKER_USER',
	            passwordVariable: 'DOCKER_PASS'
	        )]) {

        	    sh '''
	            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

        	    docker push imadsabri01/movie-service:latest
	            docker push imadsabri01/cast-service:latest

	            docker logout
	            '''
	        }
	    }
	}	

    }
}
