pipeline {
    agent any

    environment {
        // Docker Configuration
        DOCKER_IMAGE     = 'mborham6/library-app'
        DOCKER_TAG       = 'latest'
        DOCKERFILE_PATH  = 'src/Dockerfile'  
        
        // AWS/EKS Configuration
        AWS_REGION      = 'eu-west-1'
        EKS_CLUSTER     = 'flask-app-cluster'
        K8S_MANIFEST_DIR = 'k8s'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/MuhammadBorham/Library-management-system.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build from root directory with Dockerfile in src/
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", "-f ${DOCKERFILE_PATH} src/")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'new-docker-credential') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWScred'
                ]]) {
                    script {
                        sh """
                        # Configure kubectl
                        aws eks update-kubeconfig \
                            --name ${EKS_CLUSTER} \
                            --region ${AWS_REGION}

                        # Deploy Kubernetes manifests
                        kubectl apply -f ${K8S_MANIFEST_DIR}/namespace.yaml
                        kubectl apply -f ${K8S_MANIFEST_DIR}/deployment.yaml
                        kubectl apply -f ${K8S_MANIFEST_DIR}/service.yaml
                        
                        # Verify deployment
                        kubectl rollout status deployment/flask-app --namespace library
                        """
                    }
                }
            }
        }
    }

   
}
