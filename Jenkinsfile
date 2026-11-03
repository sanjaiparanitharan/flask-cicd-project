pipeline {
    agent any

    environment {
        IMAGE_NAME = 'sanjai1412/flask-cicd-app'
    }

    stages {

        // Stage 1: Get source code from GitHub
        stage('Checkout') {
            steps {
                echo 'Checking out Flask application from GitHub'
                checkout scm
            }
        }

        // Stage 2: Run application tests
        stage('Test') {
            steps {
                echo 'Running Flask application tests'

                sh '''
                    docker run --rm \
                    -v "$WORKSPACE:/app" \
                    -w /app \
                    python:3.12-slim \
                    sh -c "pip install --no-cache-dir -r requirements.txt && pytest -v"
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

                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '================================'
            echo 'CI/CD Pipeline completed successfully!'
            echo '================================'
        }

        failure {
            echo 'Pipeline failed. Please check Console Output.'
        }
    }
}