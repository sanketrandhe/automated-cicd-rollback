
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sanketrandhe/automated-cicd-rollback'
        DEPLOY_HOST = '172.31.3.206'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test Application') {
            steps {
                sh 'python3 -m py_compile app/app.py'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker build -t "$DOCKER_IMAGE:$BUILD_NUMBER" .
                        docker push "$DOCKER_IMAGE:$BUILD_NUMBER"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy and Health Check') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deployment-ssh-key',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -i "$SSH_KEY" \
                            -o StrictHostKeyChecking=accept-new \
                            "$SSH_USER@$DEPLOY_HOST" \
                            "bash -s '$DOCKER_IMAGE:$BUILD_NUMBER' '$BUILD_NUMBER'" <<'REMOTE'

                        NEW_IMAGE="$1"
                        VERSION="$2"

                        # Capture previous image and application version
                        OLD_IMAGE=$(docker inspect \
                            --format='{{.Config.Image}}' cicd-app 2>/dev/null || true)

                        OLD_VERSION=$(docker inspect \
                            --format='{{range .Config.Env}}{{println .}}{{end}}' cicd-app 2>/dev/null \
                            | grep '^APP_VERSION=' | cut -d= -f2- || true)

                        echo "Previous image: $OLD_IMAGE"
                        echo "Previous version: $OLD_VERSION"

                        # Pull and deploy new image
                        docker pull "$NEW_IMAGE"
                        docker rm -f cicd-app 2>/dev/null || true

                        docker run -d \
                            --name cicd-app \
                            --restart always \
                            -p 5000:5000 \
                            -e APP_VERSION="$VERSION" \
                            "$NEW_IMAGE"

                        sleep 5

                        # Verify application health
                        if curl -fsS http://localhost:5000/health; then
                            echo "Deployment successful!"
                        else
                            echo "Health check failed. Rolling back..."

                            docker logs cicd-app || true
                            docker rm -f cicd-app || true

                            if [ -n "$OLD_IMAGE" ]; then
                                docker run -d \
                                    --name cicd-app \
                                    --restart always \
                                    -p 5000:5000 \
                                    -e APP_VERSION="${OLD_VERSION:-1.0}" \
                                    "$OLD_IMAGE"

                                echo "Rollback completed."
                            else
                                echo "No previous image available."
                            fi

                            exit 1
                        fi

REMOTE
                    '''
                }
            }
        }
    }
}
