
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

        stage('Prepare') {
            steps {
                script {
                    env.IMAGE_TAG = env.TAG_NAME ?: env.BUILD_NUMBER
                    echo "Docker image: ${env.DOCKER_IMAGE}:${env.IMAGE_TAG}"
                }
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    test -f Dockerfile
                    test -f index.html
                    echo "Files are OK"
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
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

                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}

                        if [ -z "${TAG_NAME:-}" ]; then
                            docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest
                            docker push ${DOCKER_IMAGE}:latest
                        fi

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                buildingTag()
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-diploma']) {
                    sh '''
                        kubectl set image deployment/nginx-test \
                            nginx=${DOCKER_IMAGE}:${IMAGE_TAG} \
                            -n diploma

                        kubectl rollout status deployment/nginx-test \
                            -n diploma --timeout=180s
                    '''
                }
            }
        }
    }
}
