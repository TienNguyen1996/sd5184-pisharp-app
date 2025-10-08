pipeline {
    agent any

    environment {
        TAG = "${env.BUILD_ID}"
        AWS_REGION = "us-east-1" // Change to your region
        ECR_REPO_FRONTEND = "your-account-id.dkr.ecr.${AWS_REGION}.amazonaws.com/frontend"
        ECR_REPO_BACKEND = "your-account-id.dkr.ecr.${AWS_REGION}.amazonaws.com/backend"
        AWS_CREDENTIALS_ID = "aws-ecr-creds" // Jenkins credentials ID for AWS
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_FRONTEND}
                    """
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh """
                docker build -t ${ECR_REPO_FRONTEND}:${TAG} -t ${ECR_REPO_FRONTEND}:latest src/frontend
                """
            }
        }

        stage('Scan Frontend Image') {
            steps {
                sh """
                trivy image --severity HIGH,CRITICAL --exit-code 0 --format table ${ECR_REPO_FRONTEND}:${TAG} || true
                """
            }
        }

        stage('Build Backend Image') {
            steps {
                sh """
                docker build -t ${ECR_REPO_BACKEND}:${TAG} -t ${ECR_REPO_BACKEND}:latest src/backend
                """
            }
        }

        stage('Scan Backend Image') {
            steps {
                sh """
                trivy image --severity HIGH,CRITICAL --exit-code 0 --format table ${ECR_REPO_BACKEND}:${TAG} || true
                """
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh """
                docker push ${ECR_REPO_FRONTEND}:${TAG}
                docker push ${ECR_REPO_FRONTEND}:latest
                """
            }
        }

        stage('Push Backend Image') {
            steps {
                sh """
                docker push ${ECR_REPO_BACKEND}:${TAG}
                docker push ${ECR_REPO_BACKEND}:latest
                """
            }
        }
    }
}
