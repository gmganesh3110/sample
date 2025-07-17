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
                    sh '''
                        # Start Minikube if not running
                        if ! minikube status | grep -q "host: Running"; then
                            echo "Minikube is not running. Starting..."
                            minikube start
                        else
                            echo "Minikube is already running."
                        fi

                        # Wait until API server is responsive
                        echo "Waiting for Minikube API server to become reachable..."
                        for i in {1..10}; do
                            MINIKUBE_IP=$(minikube ip)
                            if curl -k --silent https://$MINIKUBE_IP:8443/version > /dev/null; then
                                echo "Minikube API is reachable."
                                break
                            fi
                            echo "Minikube API not ready yet. Retrying in 5s..."
                            sleep 5
                        done
                    '''

                    withCredentials([string(credentialsId: 'minikube-jenkins-token', variable: 'K8S_TOKEN')]) {
                        sh """
                            minikube start
                            kubectl apply -f deployment.yaml
                            kubectl apply -f service.yaml
                            kubectl get pods 
                        """
                    }
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