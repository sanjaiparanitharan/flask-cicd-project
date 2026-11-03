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