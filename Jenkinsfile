
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'devops-web'
        CONTAINER_NAME = 'devops-web-ci'
        NEW_CONTAINER_NAME = 'devops-web-new'
        OLD_CONTAINER_NAME = 'devops-web-ci-rollback'
        HOST_PORT = '8080'
        CONTAINER_PORT = '8080'
    }

    stages {
        stage('Test') {
            steps {
                echo 'Checking project files...'
                sh 'pwd'
                sh 'ls -la'
                sh 'test -f Dockerfile'
                sh 'test -f app/index.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -eu

                    OLD_EXISTS=0

                    if docker container inspect "$OLD_CONTAINER_NAME" >/dev/null 2>&1; then
                        echo "Rollback container already exists. Stop deployment."
                        exit 1
                    fi

                    docker rm -f "$NEW_CONTAINER_NAME" >/dev/null 2>&1 || true

                    if docker container inspect "$CONTAINER_NAME" >/dev/null 2>&1; then
                        OLD_EXISTS=1
                        docker stop "$CONTAINER_NAME"
                        docker rename "$CONTAINER_NAME" "$OLD_CONTAINER_NAME"
                    fi

                    if ! docker run -d \
                        --name "$NEW_CONTAINER_NAME" \
                        -p "$HOST_PORT:$CONTAINER_PORT" \
                        "$IMAGE_NAME:$BUILD_NUMBER"; then

                        echo "New container failed to start."

                        if [ "$OLD_EXISTS" -eq 1 ]; then
                            docker rename "$OLD_CONTAINER_NAME" "$CONTAINER_NAME"
                            docker start "$CONTAINER_NAME"
                        fi

                        exit 1
                    fi

                    if ! curl -fsS --retry 10 --retry-all-errors --retry-delay 2 \
                        "http://localhost:$HOST_PORT/" >/dev/null; then

                        echo "Health check failed. Rolling back."

                        docker rm -f "$NEW_CONTAINER_NAME" >/dev/null 2>&1 || true

                        if [ "$OLD_EXISTS" -eq 1 ]; then
                            docker rename "$OLD_CONTAINER_NAME" "$CONTAINER_NAME"
                            docker start "$CONTAINER_NAME"
                        fi

                        exit 1
                    fi

                    docker rename "$NEW_CONTAINER_NAME" "$CONTAINER_NAME"

                    if [ "$OLD_EXISTS" -eq 1 ]; then
                        docker rm -f "$OLD_CONTAINER_NAME"
                    fi

                    echo "Deployment successful."
                '''
            }
        }
    }
}

