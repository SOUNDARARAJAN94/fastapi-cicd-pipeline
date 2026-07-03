pipeline {
    agent any

    environment {
        AWS_REGION = "ap-southeast-2"
        AWS_ACCOUNT_ID = "824604427937"

        BACKEND_REPO = "fastapi-backend"
        FRONTEND_REPO = "fastapi-frontend"

        BACKEND_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}:latest"
        FRONTEND_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                sh 'docker build -t fastapi-backend:latest -f backend/Dockerfile .'
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                sh 'docker build -t fastapi-frontend:latest -f frontend/Dockerfile --build-arg VITE_API_URL=http://localhost .'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag Images') {
            steps {
                sh '''
                docker tag fastapi-backend:latest $BACKEND_IMAGE
                docker tag fastapi-frontend:latest $FRONTEND_IMAGE
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                docker push $BACKEND_IMAGE
                docker push $FRONTEND_IMAGE
                '''
            }
        }
    }
}
