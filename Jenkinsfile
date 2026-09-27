pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID = '502768730616'
        AWS_REGION     = 'us-east-1'
        REGISTRY       = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        VERSION        = '1.0.0'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('AWS ECR Login') {
            steps {
                // Authenticates Jenkins docker client against AWS ECR
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${REGISTRY}"
            }
        }
        stage('Build & Push Microservices') {
            steps {
                script {
                    def services = [
                        'streaming-auth': 'backend/authService',
                        'streaming-stream': '-f backend/streamingService/Dockerfile backend',
                        'streaming-admin': '-f backend/adminService/Dockerfile backend',
                        'streaming-chat': '-f backend/chatService/Dockerfile backend',
                        'streaming-frontend': 'frontend'
                    ]
                    
                    services.each { ecrRepo, buildContext ->
                        echo "Building image for ${ecrRepo}..."
                        sh "docker build -t ${REGISTRY}/${ecrRepo}:${VERSION} ${buildContext}"
                        
                        echo "Pushing image for ${ecrRepo}..."
                        sh "docker push ${REGISTRY}/${ecrRepo}:${VERSION}"
                    }
                }
            }
        }
        stage('Deploy via Helm') {
            steps {
                // Deploys the application directly into your live EKS cluster
                sh "aws eks update-kubeconfig --region ${AWS_REGION} --name streaming-cluster"
                sh "helm upgrade --install streamingapp ./streamingapp --namespace streaming --create-namespace"
            }
        }
    }
}
