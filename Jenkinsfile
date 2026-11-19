
pipeline {
    agent any

    parameters {
        string(
            name: 'EC2_HOST',
            defaultValue: '',
            description: 'AWS EC2 Public IP'
        )
    }

    environment {
        IMAGE_NAME = 'flask-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'python3 --version'
                sh 'python3 -m compileall -q app.py'
            }
        }

        stage('Test') {
            steps {
                sh 'echo Application syntax test passed'
            }
        }

        stage('Docker Build and Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin

                        docker build -t "$DH_USER/$IMAGE_NAME:$BUILD_NUMBER" .
                        docker push "$DH_USER/$IMAGE_NAME:$BUILD_NUMBER"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to AWS') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    ),
                    file(
                        credentialsId: 'aws-ec2-key',
                        variable: 'SSH_KEY'
                    )
                ]) {
                    sh '''
                        ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ubuntu@$EC2_HOST "sudo docker pull $DH_USER/flask-app:$BUILD_NUMBER"

                        ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ubuntu@$EC2_HOST "sudo docker rm -f flask-app || true"

                        ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ubuntu@$EC2_HOST "sudo docker run -d --restart unless-stopped --name flask-app -p 80:5000 $DH_USER/flask-app:$BUILD_NUMBER"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline Completed Successfully!'
        }

        failure {
            echo 'CI/CD Pipeline Failed!'
        }
    }
}
