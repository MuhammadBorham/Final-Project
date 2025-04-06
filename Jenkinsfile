pipeline {
    agent any

    environment {
        // Docker Configuration
        DOCKER_IMAGE     = 'mborham6/jenkins-flask'
        DOCKER_TAG       = 'latest'
        DOCKERFILE_PATH  = 'src/Dockerfile'  // Path to Dockerfile
        
        // AWS/EKS Configuration
        AWS_REGION      = 'eu-west-1'
        EKS_CLUSTER     = 'flask-app-cluster'
        K8S_MANIFEST_DIR = 'k8s'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/MuhammadBorham/Final-Project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build from root directory with Dockerfile in src/
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", "-f ${DOCKERFILE_PATH} .")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
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
                        # Configure kubectl
                        aws eks update-kubeconfig \
                            --name ${EKS_CLUSTER} \
                            --region ${AWS_REGION}

                        # Deploy Kubernetes manifests
                        kubectl apply -f ${K8S_MANIFEST_DIR}
                        
                        # Verify deployment
                        kubectl rollout status deployment/flask-app
                        """
                    }
                }
            }
        }
    }

   
}
