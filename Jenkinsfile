pipeline {
    agent any
    environment {
        // 1. Paste your AWS Keys here so Jenkins can authenticate
        AWS_ACCESS_KEY_ID     = 'ASIAXKD22AH4KWTDJGT2'
        AWS_SECRET_ACCESS_KEY = 'tGzocWcPqL1HqBNnyxeCPEcmMg5DBtOuM/BtIYly'
        
        // Your account configurations
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
                sh "aws eks update-kubeconfig --region ${AWS_REGION} --name streaming-cluster"
                sh "helm upgrade --install streamingapp ./streamingapp --namespace streaming --create-namespace"
            }
        }
    }
}
