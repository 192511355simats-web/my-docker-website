pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        IMAGE_NAME = 'ramprasadh-portfolio'
        CONTAINER_NAME = 'ramprasadh-site'
        SITE_PORT = '8081'
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'git config --global --add safe.directory "$WORKSPACE"'
                checkout scm
            }
        }

        stage('Build image') {
            steps {
                sh 'docker build --tag "$IMAGE_NAME:$BUILD_NUMBER" --tag "$IMAGE_NAME:latest" .'
            }
        }

        stage('Test image') {
            steps {
                sh '''
                    set -eu
                    docker rm --force ramprasadh-test >/dev/null 2>&1 || true
                    docker run --detach --name ramprasadh-test "$IMAGE_NAME:$BUILD_NUMBER"
                    trap 'docker rm --force ramprasadh-test >/dev/null 2>&1 || true' EXIT
                    docker exec ramprasadh-test wget -qO- http://127.0.0.1/ | grep -qi '<html'
                '''
            }
        }

        stage('Deploy to localhost') {
            steps {
                sh '''
                    set -eu
                    docker rm --force "$CONTAINER_NAME" >/dev/null 2>&1 || true
                    docker run --detach --restart unless-stopped --name "$CONTAINER_NAME" --publish "$SITE_PORT:80" "$IMAGE_NAME:$BUILD_NUMBER"
                    docker exec "$CONTAINER_NAME" wget -qO- http://127.0.0.1/ | grep -qi '<html'
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded: http://localhost:8081'
        }
        failure {
            echo 'Pipeline failed. Review the stage output; the last deployed site may still be available.'
        }
    }
}
