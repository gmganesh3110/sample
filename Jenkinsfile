pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME = 'sample'
        IMAGE_TAG = '1.0'
        DOCKER_HUB_REPO = 'gmganesh'
        // Removed KUBECONFIG as we're using token auth instead
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }
        
        stage('Tag Docker Image') {
            steps {
                script {
                    sh "docker tag ${IMAGE_NAME} ${DOCKER_HUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        
        stage('Login to Docker Hub') {
            steps {
                script {
                    sh "echo \$DOCKER_HUB_CREDENTIALS_PSW | docker login -u \$DOCKER_HUB_CREDENTIALS_USR --password-stdin"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "docker push ${DOCKER_HUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        
        stage('Configure Kubernetes Access') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'minikube-token', variable: 'K8S_TOKEN')]) {
                        sh """
                            # Get current Minikube IP
                            MINIKUBE_IP=\$(minikube ip)
                            
                            # Configure kubectl access
                            kubectl config set-cluster minikube \
                              --server=https://\${MINIKUBE_IP}:8443 \
                              --insecure-skip-tls-verify=true
                            kubectl config set-credentials jenkins \
                              --token=${K8S_TOKEN}
                            kubectl config set-context minikube \
                              --cluster=minikube \
                              --user=jenkins
                            kubectl config use-context minikube
                        """
                    }
                }
            }
        }
        
        stage('Deploy to Minikube') {
            steps {
                script {
                    sh """
                        kubectl apply -f deployment.yaml --validate=false
                        kubectl apply -f service.yaml --validate=false
                    """
                }
            }
        }
    
        stage('Verify Deployment') {
            steps {
                script {
                    sh "kubectl rollout status deployment/sample-deployment"
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Clean up Docker credentials
                sh 'docker logout'
                
                // Clean up built images to save space
                sh "docker rmi ${IMAGE_NAME} ${DOCKER_HUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG} || true"
                
                // Clean up kubectl config (now stored in memory, no file to clean)
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}