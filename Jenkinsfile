pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '700800569732'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        AUTH_IMAGE = "${ECR_REGISTRY}/streaming-auth"
        STREAM_IMAGE = "${ECR_REGISTRY}/streaming-stream"
        ADMIN_IMAGE = "${ECR_REGISTRY}/streaming-admin"
        CHAT_IMAGE = "${ECR_REGISTRY}/streaming-chat"
        FRONTEND_IMAGE = "${ECR_REGISTRY}/streaming-frontend"

        IMAGE_TAG = '1.0.0'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out StreamingApp source code...'
                checkout scm
            }
        }

        stage('AWS Identity') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-ecr-credentials-sadik']
                ]) {
                    sh '''
                        echo "Checking AWS identity..."
                        aws sts get-caller-identity
                    '''
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-ecr-credentials-sadik']
                ]) {
                    sh '''
                        echo "Logging in to Amazon ECR..."

                        aws ecr get-login-password \
                          --region ${AWS_REGION} | \
                        docker login \
                          --username AWS \
                          --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

stage('Build Docker Images') {
    steps {
        sh '''
            echo "Building Auth Service..."
            docker build \
              -t ${AUTH_IMAGE}:${IMAGE_TAG} \
              -f backend/authService/Dockerfile \
              backend/authService

            echo "Building Streaming Service..."
            docker build \
              -t ${STREAM_IMAGE}:${IMAGE_TAG} \
              -f backend/streamingService/Dockerfile \
              backend/streamingService

            echo "Building Admin Service..."
            docker build \
              -t ${ADMIN_IMAGE}:${IMAGE_TAG} \
              -f backend/adminService/Dockerfile \
              backend/adminService

            echo "Building Chat Service..."
            docker build \
              -t ${CHAT_IMAGE}:${IMAGE_TAG} \
              -f backend/chatService/Dockerfile \
              backend/chatService

            echo "Building Frontend..."
            docker build \
              -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
              -f frontend/Dockerfile \
              frontend

            echo "=========================================="
            echo "All Docker images built successfully."
            echo "=========================================="

            docker images | grep streaming-
        '''
    }
}

        stage('Push Images to ECR') {
            steps {
                sh '''
                    echo "Pushing Auth Service..."
                    docker push ${AUTH_IMAGE}:${IMAGE_TAG}

                    echo "Pushing Streaming Service..."
                    docker push ${STREAM_IMAGE}:${IMAGE_TAG}

                    echo "Pushing Admin Service..."
                    docker push ${ADMIN_IMAGE}:${IMAGE_TAG}

                    echo "Pushing Chat Service..."
                    docker push ${CHAT_IMAGE}:${IMAGE_TAG}

                    echo "Pushing Frontend..."
                    docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}

                    echo "All images pushed successfully."
                '''
            }
        }

        stage('Verify ECR Images') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-ecr-credentials-sadik']
                ]) {
                    sh '''
                        echo "Verifying images in ECR..."

                        aws ecr describe-images \
                          --repository-name streaming-auth \
                          --region ${AWS_REGION} \
                          --query 'imageDetails[*].imageTags' \
                          --output table

                        aws ecr describe-images \
                          --repository-name streaming-stream \
                          --region ${AWS_REGION} \
                          --query 'imageDetails[*].imageTags' \
                          --output table

                        aws ecr describe-images \
                          --repository-name streaming-admin \
                          --region ${AWS_REGION} \
                          --query 'imageDetails[*].imageTags' \
                          --output table

                        aws ecr describe-images \
                          --repository-name streaming-chat \
                          --region ${AWS_REGION} \
                          --query 'imageDetails[*].imageTags' \
                          --output table

                        aws ecr describe-images \
                          --repository-name streaming-frontend \
                          --region ${AWS_REGION} \
                          --query 'imageDetails[*].imageTags' \
                          --output table
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '=============================================='
            echo 'StreamingApp CI Pipeline completed SUCCESSFULLY'
            echo 'All 5 Docker images pushed to ECR.'
            echo '=============================================='
        }

        failure {
            echo '=============================================='
            echo 'StreamingApp CI Pipeline FAILED'
            echo 'Please check the Jenkins console output.'
            echo '=============================================='
        }
    }
}