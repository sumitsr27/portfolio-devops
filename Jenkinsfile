pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'portfolio'
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_USERNAME = 'sumitsr2003'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        } 
        ///wer
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                                                    passwordVariable: 'DOCKER_PASSWORD', 
                                                    usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        sh "docker push ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        sh "docker tag ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_USERNAME}/${DOCKER_IMAGE}:latest"
                        sh "docker push ${DOCKER_USERNAME}/${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
    steps {
        script {
            // For macOS compatibility, provide empty backup extension with -i ''
            sh "sed -i '' 's|image: .*|image: ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}|' k8s-deployment.yaml"
            
            // Apply Kubernetes configurations
            sh "kubectl apply -f k8s-deployment.yaml"
            sh "kubectl apply -f k8s-service.yaml"
            
            // Wait for deployment to roll out
            sh "kubectl rollout status deployment/portfolio-app"
        }
    }
}
    }
    //hi123
    post {
        always {
            // Clean up Docker images
            sh "docker rmi ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG} || true"
            sh "docker rmi ${DOCKER_USERNAME}/${DOCKER_IMAGE}:latest || true"
        }
    }
} 