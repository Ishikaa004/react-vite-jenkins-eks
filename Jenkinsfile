pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        IMAGE_NAME = 'admin/react-vite-jenkins-app'
        TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install --legacy-peer-deps'
            }
        }
        stage('Build React Application') {
            steps {
                sh 'npm run build'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} -t ${IMAGE_NAME}:latest ."
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        sh "docker push ${IMAGE_NAME}:${TAG}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }
        stage('Update Kubernetes Deployment') {
            steps {
                sh "sed -i 's|image:.*|image: ${IMAGE_NAME}:${TAG}|g' k8s-deployment.yaml"
                sh 'kubectl apply -f k8s-deployment.yaml'
            }
        }
        stage('Verify Deployment Status') {
            steps {
                sh 'kubectl rollout status deployment/react-vite-app'
                sh 'kubectl get pods'
                sh 'kubectl get svc'
            }
        }
    }
}
