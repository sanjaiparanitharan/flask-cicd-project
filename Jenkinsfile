
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
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
                echo 'Deploying Flask application to EC2'

                sh '''
                    docker pull ${IMAGE_NAME}:latest

                    docker rm -f flask-cicd-app || true

                    docker run -d \
                        --name flask-cicd-app \
                        --restart unless-stopped \
                        -p 5000:5000 \
                        ${IMAGE_NAME}:latest

                    sleep 5

                    docker ps --filter name=flask-cicd-app

                    curl --fail http://localhost:5000/health
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY!'
            echo '======================================'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Console Output.'
        }
    }
}
