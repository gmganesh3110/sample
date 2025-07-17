pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME = 'sample'
        IMAGE_TAG = '1.0'
        DOCKER_HUB_REPO = 'gmganesh'
        KUBECONFIG = '/home/jenkins/.kube/config' // Path to your kubeconfig for Minikube
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gmganesh3110/sample.git'
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
                    // Update the deployment with the new image
                    sh "kubectl set image deployment/sample-deployment sample=${DOCKER_HUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG} --record"
                    
                    // Alternatively, if you want to apply the entire deployment YAML:
                    // sh "kubectl apply -f deployment.yaml"
                    // sh "kubectl apply -f service.yaml"
                    
                    // Verify deployment
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