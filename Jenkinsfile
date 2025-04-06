pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mborham6/library-app"
        DOCKER_TAG = "latest"
        EKS_CLUSTER = "flask-app-cluster"
        AWS_REGION = "eu-west-1"
    }

    stages {
        // Stage 1: Checkout code from Git
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MuhammadBorham/Final-Project.git'
            }
        }

        // Stage 2: Build Docker image
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        // Stage 3: Push to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        // Stage 4: Deploy to EKS
        stage('Deploy to EKS') {
            steps {
                script {
                    // Authenticate with AWS EKS
                    sh "aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER}"

                    // Apply Kubernetes manifests
                    sh "kubectl apply -f kubernetes/deployment.yaml"
                    sh "kubectl apply -f kubernetes/service.yaml"
                    sh "kubectl apply -f kubernetes/ingress.yaml"  // Optional

                    // Verify deployment
                    sh "kubectl rollout status deployment/library-app"
                }
            }
        }
    }

    post {
        success {
            slackSend channel: '#devops', message: "✅ Successfully deployed ${DOCKER_IMAGE}:${DOCKER_TAG} to EKS"
        }
        failure {
            slackSend channel: '#devops', message: "❌ Failed to deploy ${DOCKER_IMAGE}:${DOCKER_TAG} to EKS"
        }
    }
}
