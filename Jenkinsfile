pipeline {
    agent any

    environment {
        REPOSITORY = 'webapp'
        IMAGE_NAME = 'webapp'
        IMAGE_TAG = "${BUILD_NUMBER}"
        USERNAME = 'jaymahakal123'
    }

    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Docker Cleanup") {
            steps {
                sh '''
                    echo "Cleaning unused Docker resources..."
                    docker system prune -af
                '''
            }
        }

        stage("Build") {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage("DockerHub Login and Push") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        printf '%s' "$DOCKERHUB_TOKEN" |
                            docker login -u "$DOCKERHUB_USERNAME" --password-stdin

                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker push \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage("Deploy") {
            steps {
                sh '''
                    docker ps -q --filter publish=80 | xargs -r docker rm -f
                    docker rm -f webapp-container 2>/dev/null || true

                    docker run -d --name webapp-container --restart unless-stopped \
                        -p 80:80 ${IMAGE_NAME}:${IMAGE_TAG}

                    sleep 5
                    [ "$(docker inspect -f '{{.State.Running}}' webapp-container)" = "true" ] \
                        || { docker logs webapp-container; exit 1; }
                    echo "Deployment successful!"
                '''
            }
        }
    }
}