
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'zhelonkindd/nginx-demo'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    test -f Dockerfile
                    test -f index.html
                    echo "Files is ok"
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'f0d4c677-55d1-4f1c-9a03-35d1a29e1fb0',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'YEEEP! Docker image successfully published!'
        }
        failure {
            echo 'AWWW! Pipeline failed!'
        }
    }
}
