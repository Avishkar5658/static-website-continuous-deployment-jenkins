pipeline {
    agent any

    triggers {
        githubPush()
    }

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'my-static-website'
        CONTAINER_NAME = 'static-website'
        HOST_PORT = '8081'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    set -eu
                    test -f Dockerfile
                    test -f index.html
                    echo "Website files verified."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Deploy Website') {
            steps {
                sh '''
                    set -eu

                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p ${HOST_PORT}:80 \
                      ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu
                    docker ps --filter "name=${CONTAINER_NAME}"
                    curl --fail --retry 5 --retry-delay 2 \
                      http://127.0.0.1:${HOST_PORT}/
                    echo "Website deployed successfully."
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
            echo 'Open http://YOUR-EC2-PUBLIC-IP:8081'
        }

        failure {
            echo 'Deployment failed. Check the Jenkins Console Output.'
        }
    }
}
