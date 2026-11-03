pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        IMAGE_NAME = 'sanjai1412/flask-cicd-app'
    }

    stages {

        // Stage 1: Checkout source code
        stage('Checkout') {
            steps {
                echo 'Checking out Flask application from GitHub'
                checkout scm
            }
        }

        // Stage 2: Run Flask application tests
        stage('Test') {
            steps {
                echo 'Running Flask application tests'

                sh '''
                    /opt/flask-ci/venv/bin/pytest -v
                '''
            }
        }

        // Stage 3: Build Docker image
        stage('Docker Build') {
            steps {
                echo 'Building Docker image'

                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                '''
            }
        }

        // Stage 4: Push image to Docker Hub
        stage('Docker Push') {
            steps {
                echo 'Pushing Docker image to Docker Hub'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        trap 'docker logout >/dev/null 2>&1 || true' EXIT

                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
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