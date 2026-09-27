pipeline {
    agent { label 'linux-manik' }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '845041271182.dkr.ecr.ap-south-1.amazonaws.com'

        AUTH_IMAGE = 'streamingapp-auth'
        STREAMING_IMAGE = 'streamingapp-streaming'
        ADMIN_IMAGE = 'streamingapp-admin'
        CHAT_IMAGE = 'streamingapp-chat'
        FRONTEND_IMAGE = 'streamingapp-frontend'

        // Frontend API URLs for the CI image.
        // These can be updated later when the EKS/Ingress URLs are known.
        AUTH_API_URL = 'http://localhost:3001'
        STREAMING_API_URL = 'http://localhost:3002'
        STREAMING_PUBLIC_URL = 'http://localhost:3002'
        ADMIN_API_URL = 'http://localhost:3003'
        CHAT_API_URL = 'http://localhost:3004'
        CHAT_SOCKET_URL = 'http://localhost:3004'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare') {
            steps {
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Building images with tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    set -e

                    docker build \
                      -t ${AUTH_IMAGE}:${IMAGE_TAG} \
                      -t ${AUTH_IMAGE}:latest \
                      ./backend/authService

                    docker build \
                      -t ${STREAMING_IMAGE}:${IMAGE_TAG} \
                      -t ${STREAMING_IMAGE}:latest \
                      -f ./backend/streamingService/Dockerfile \
                      ./backend

                    docker build \
                      -t ${ADMIN_IMAGE}:${IMAGE_TAG} \
                      -t ${ADMIN_IMAGE}:latest \
                      -f ./backend/adminService/Dockerfile \
                      ./backend

                    docker build \
                      -t ${CHAT_IMAGE}:${IMAGE_TAG} \
                      -t ${CHAT_IMAGE}:latest \
                      -f ./backend/chatService/Dockerfile \
                      ./backend

                    docker build \
                      --build-arg REACT_APP_AUTH_API_URL=${AUTH_API_URL} \
                      --build-arg REACT_APP_STREAMING_API_URL=${STREAMING_API_URL} \
                      --build-arg REACT_APP_STREAMING_PUBLIC_URL=${STREAMING_PUBLIC_URL} \
                      --build-arg REACT_APP_ADMIN_API_URL=${ADMIN_API_URL} \
                      --build-arg REACT_APP_CHAT_API_URL=${CHAT_API_URL} \
                      --build-arg REACT_APP_CHAT_SOCKET_URL=${CHAT_SOCKET_URL} \
                      -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                      -t ${FRONTEND_IMAGE}:latest \
                      ./frontend
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-ecr-credentials-2']
                ]) {
                    sh '''
                        set -e

                        aws sts get-caller-identity

                        aws ecr get-login-password \
                          --region ${AWS_REGION} \
                          | docker login \
                            --username AWS \
                            --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Tag Images for ECR') {
            steps {
                sh '''
                    set -e

                    docker tag ${AUTH_IMAGE}:${IMAGE_TAG} \
                      ${ECR_REGISTRY}/${AUTH_IMAGE}:${IMAGE_TAG}
                    docker tag ${AUTH_IMAGE}:latest \
                      ${ECR_REGISTRY}/${AUTH_IMAGE}:latest

                    docker tag ${STREAMING_IMAGE}:${IMAGE_TAG} \
                      ${ECR_REGISTRY}/${STREAMING_IMAGE}:${IMAGE_TAG}
                    docker tag ${STREAMING_IMAGE}:latest \
                      ${ECR_REGISTRY}/${STREAMING_IMAGE}:latest

                    docker tag ${ADMIN_IMAGE}:${IMAGE_TAG} \
                      ${ECR_REGISTRY}/${ADMIN_IMAGE}:${IMAGE_TAG}
                    docker tag ${ADMIN_IMAGE}:latest \
                      ${ECR_REGISTRY}/${ADMIN_IMAGE}:latest

                    docker tag ${CHAT_IMAGE}:${IMAGE_TAG} \
                      ${ECR_REGISTRY}/${CHAT_IMAGE}:${IMAGE_TAG}
                    docker tag ${CHAT_IMAGE}:latest \
                      ${ECR_REGISTRY}/${CHAT_IMAGE}:latest

                    docker tag ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                      ${ECR_REGISTRY}/${FRONTEND_IMAGE}:${IMAGE_TAG}
                    docker tag ${FRONTEND_IMAGE}:latest \
                      ${ECR_REGISTRY}/${FRONTEND_IMAGE}:latest
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    set -e

                    docker push ${ECR_REGISTRY}/${AUTH_IMAGE}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${AUTH_IMAGE}:latest

                    docker push ${ECR_REGISTRY}/${STREAMING_IMAGE}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${STREAMING_IMAGE}:latest

                    docker push ${ECR_REGISTRY}/${ADMIN_IMAGE}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${ADMIN_IMAGE}:latest

                    docker push ${ECR_REGISTRY}/${CHAT_IMAGE}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${CHAT_IMAGE}:latest

                    docker push ${ECR_REGISTRY}/${FRONTEND_IMAGE}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${FRONTEND_IMAGE}:latest
                '''
            }
        }
    }

    post {
        success {
            echo "CI pipeline completed successfully."
            echo "Images pushed to ECR with tag: ${IMAGE_TAG}"
        }

        failure {
            echo "CI pipeline failed. Check the stage logs above."
        }
    }
}