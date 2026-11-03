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
