pipeline {

    agent any

    environment {
        DOCKER_USER = "imadsabri01"
        MOVIE_IMAGE = "movie-service"
        CAST_IMAGE = "cast-service"
        IMAGE_TAG = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Verify Workspace') {
            steps {
                sh '''
                pwd
                ls -la
                '''
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
                docker build -t $DOCKER_USER/$MOVIE_IMAGE:$IMAGE_TAG ./movie-service
                docker build -t $DOCKER_USER/$CAST_IMAGE:$IMAGE_TAG ./cast-service
                '''
            }
        }

        stage('Push Images') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_LOGIN',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_LOGIN" --password-stdin

                    docker push $DOCKER_USER/$MOVIE_IMAGE:$IMAGE_TAG
                    docker push $DOCKER_USER/$CAST_IMAGE:$IMAGE_TAG

                    docker logout
                    '''
                }
            }
        }

        stage('Deploy DEV') {

            environment {
                KUBECONFIG = credentials('config.txt')
            }

            steps {

                sh '''
                mkdir -p .kube
                cp $KUBECONFIG .kube/config

                helm upgrade --install movie-release ./charts \
                --namespace dev \
                --set image.repository=$DOCKER_USER/$MOVIE_IMAGE \
                --set image.tag=$IMAGE_TAG
                '''
            }
        }

        stage('Deploy QA') {

            environment {
                KUBECONFIG = credentials('config.txt')
            }

            steps {

                sh '''
                mkdir -p .kube
                cp $KUBECONFIG .kube/config

                helm upgrade --install movie-release ./charts \
                --namespace qa \
                --set image.repository=$DOCKER_USER/$MOVIE_IMAGE \
                --set image.tag=$IMAGE_TAG
                '''
            }
        }

        stage('Deploy STAGING') {

            environment {
                KUBECONFIG = credentials('config.txt')
            }

            steps {

                sh '''
                mkdir -p .kube
                cp $KUBECONFIG .kube/config

                helm upgrade --install movie-release ./charts \
                --namespace staging \
                --set image.repository=$DOCKER_USER/$MOVIE_IMAGE \
                --set image.tag=$IMAGE_TAG
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

            environment {
                KUBECONFIG = credentials('config.txt')
            }

            steps {

                sh '''
                mkdir -p .kube
                cp $KUBECONFIG .kube/config

                helm upgrade --install movie-release ./charts \
                --namespace prod \
                --set image.repository=$DOCKER_USER/$MOVIE_IMAGE \
                --set image.tag=$IMAGE_TAG
                '''
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}
