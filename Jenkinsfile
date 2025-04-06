pipeline {
    agent any
    
    environment {
        // AWS/EKS Config
        AWS_REGION      = "eu-west-1"
        EKS_CLUSTER     = "flask-app-cluster"
        
        // Docker Config
        DOCKER_IMAGE = 'mborham6/jenkins-flask:latest' 
        DOCKER_HUB_CREDENTIALS = credentials('new-docker-credential')
        
        // Kubernetes Config
        K8S_DIR         = "k8s"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/MuhammadBorham/Final-Project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build from root directory
                    docker.build("${DOCKER_IMAGE}", "-f Dockerfile .")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS}") {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    script {
                        sh """
                        aws eks update-kubeconfig --name ${EKS_CLUSTER} --region ${AWS_REGION}
                        kubectl apply -f ${K8S_DIR}
                        """
                    }
                }
            }
        }
    }

