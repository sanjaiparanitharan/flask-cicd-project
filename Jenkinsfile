
stage('Deploy to EC2') {
    steps {
        echo 'Deploying Flask application with rollback protection'

        sh '''
            set -e

            NEW_IMAGE="${IMAGE_NAME}:${BUILD_NUMBER}"

            # Save the currently running version's immutable image ID
            PREVIOUS_IMAGE_ID=$(docker inspect \
                --format='{{.Image}}' flask-cicd-app 2>/dev/null || true)

            echo "Previous image: ${PREVIOUS_IMAGE_ID}"
            echo "New image: ${NEW_IMAGE}"

            docker pull "$NEW_IMAGE"

            # Stop the old application
            docker rm -f flask-cicd-app || true

            # Start the new version
            docker run -d \
                --name flask-cicd-app \
                --restart unless-stopped \
                -p 5000:5000 \
                "$NEW_IMAGE"

            # Check application health
            HEALTHY=false

            for i in $(seq 1 10); do
                if curl --fail --silent http://localhost:5000/health; then
                    HEALTHY=true
                    break
                fi
                echo "Health check attempt $i failed. Retrying..."
                sleep 3
            done

            if [ "$HEALTHY" = "true" ]; then
                echo "New deployment is healthy!"
            else
                echo "New deployment failed. Starting rollback..."

                docker rm -f flask-cicd-app || true

                if [ -n "$PREVIOUS_IMAGE_ID" ]; then
                    docker run -d \
                        --name flask-cicd-app \
                        --restart unless-stopped \
                        -p 5000:5000 \
                        "$PREVIOUS_IMAGE_ID"

                    sleep 5

                    if curl --fail http://localhost:5000/health; then
                        echo "ROLLBACK SUCCESSFUL: Previous version restored."
                    else
                        echo "ROLLBACK FAILED: Previous version is unhealthy."
                        exit 1
                    fi
                else
                    echo "No previous image available for rollback."
                    exit 1
                fi

                exit 1
            fi
        '''
    }
}
