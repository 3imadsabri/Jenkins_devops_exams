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

        stage('Deploy DEV') {
            steps {
                sh '''
		helm upgrade --install movie-release ./charts \
		--namespace dev \
		--set movie.image.repository=imadsabri01/movie-service \
		--set movie.image.tag=latest \
		--set cast.image.repository=imadsabri01/cast-service \
		--set cast.image.tag=latest
                '''
            }
        }

        stage('Deploy QA') {
            steps {
                sh '''
		helm upgrade --install movie-release ./charts \
		--namespace qa \
		--set movie.image.repository=imadsabri01/movie-service \
		--set movie.image.tag=latest \
		--set cast.image.repository=imadsabri01/cast-service \
		--set cast.image.tag=latest
                '''
            }
        }

        stage('Deploy STAGING') {
            steps {
                sh '''
		helm upgrade --install movie-release ./charts \
	        --namespace staging \
	        --set movie.image.repository=imadsabri01/movie-service \
	        --set movie.image.tag=latest \
	        --set cast.image.repository=imadsabri01/cast-service \
	        --set cast.image.tag=latest
                '''
            }
        }

	stage('Manual Approval') {
 	   steps {
 	       timeout(time: 15, unit: 'MINUTES') {
        	    input(
        	        message: 'Do you want to deploy to Production?',
      		        ok: 'YES'
           	    )
 	       }
	    }
	}

	stage('Deploy PROD') {
	    steps {
	        sh '''
	        helm upgrade --install movie-release ./charts \
	        --namespace prod \
	        --set movie.image.repository=imadsabri01/movie-service \
	        --set movie.image.tag=latest \
	        --set cast.image.repository=imadsabri01/cast-service \
	        --set cast.image.tag=latest
		'''
	    }
	}
    }
}
