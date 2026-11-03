
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'sanjai1412/flask-cicd-app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out Flask application'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Checking Python application syntax'
                sh '''
                    /opt/flask-ci/venv/bin/python -m compileall -q .
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running Flask tests'
                sh '''
                    /opt/flask-ci/venv/bin/pytest -v -p no:cacheprovider
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image'
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing image to Docker Hub'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASS" | tr -d '\\r\\n' | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Deploying Flask application with rollback protection'

                sh '''
                    set -eu

                    NEW_IMAGE="${IMAGE_NAME}:${BUILD_NUMBER}"

                    PREVIOUS_IMAGE_ID=$(docker inspect \
                        --format='{{.Image}}' flask-cicd-app 2>/dev/null || true)

                    echo "Previous image: ${PREVIOUS_IMAGE_ID}"
                    echo "New image: ${NEW_IMAGE}"

                    docker pull "$NEW_IMAGE"

                    rollback() {
                        echo "Deployment failed. Starting rollback..."

                        docker rm -f flask-cicd-app || true

                        if [ -z "$PREVIOUS_IMAGE_ID" ]; then
                            echo "No previous image available!"
                            return 1
                        fi

                        docker run -d \
                            --name flask-cicd-app \
                            --restart unless-stopped \
                            -p 5000:5000 \
                            "$PREVIOUS_IMAGE_ID"

                        for i in $(seq 1 10); do
                            if curl --fail --silent http://localhost:5000/health; then
                                echo "ROLLBACK SUCCESSFUL: Previous version restored."
                                return 0
                            fi
                            sleep 3
                        done

                        echo "ROLLBACK FAILED: Previous version is unhealthy."
                        return 1
                    }

                    docker rm -f flask-cicd-app || true

                    if ! docker run -d \
                        --name flask-cicd-app \
                        --restart unless-stopped \
                        -p 5000:5000 \
                        "$NEW_IMAGE"; then

                        rollback
                        exit 1
                    fi

                    HEALTHY=false

                    for i in $(seq 1 10); do
                        if curl --fail --silent http://localhost:5000/health-fail-demo; then
                            HEALTHY=true
                            break
                        fi

                        echo "Health check attempt $i failed."
                        sleep 3
                    done

                    if [ "$HEALTHY" = "true" ]; then
                        echo "New deployment is healthy!"
                    else
                        rollback
                        exit 1
                    fi
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY!'
        }

        failure {
            echo 'CI/CD PIPELINE FAILED. Check Console Output.'
        }
    }
}
     
