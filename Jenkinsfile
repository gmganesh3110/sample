pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME = 'sample'
        IMAGE_TAG = '1.0'
        DOCKER_HUB_REPO = 'gmganesh'
        KUBECONFIG = "${WORKSPACE}/.kube/config"  // Use workspace path instead of hardcoded /home/jenkins
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
                    sh "echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin"
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
        
        stage('Deploy to Minikube') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'minikube-token', variable: 'K8S_TOKEN')]) {
                        sh """
                            kubectl config set-cluster minikube \
                            --server=https://$(minikube ip):8443 \
                            --insecure-skip-tls-verify=true
                            kubectl config set-credentials jenkins \
                            --token=${K8S_TOKEN}
                            kubectl config set-context minikube \
                            --cluster=minikube \
                            --user=jenkins
                            kubectl config use-context minikube
                            kubectl apply -f deployment.yaml
                            kubectl apply -f service.yaml
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
                
                // Optional: Clean up built images to save space
                sh "docker rmi ${IMAGE_NAME} ${DOCKER_HUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG} || true"
                
                // Clean up kubeconfig
                sh "rm -rf ${WORKSPACE}/.kube || true"
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