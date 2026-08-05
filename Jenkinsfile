pipeline {
    agent any

    environment {
        IMAGE_NAME = 'ishikaa24/react-vite-jenkins-app'
        TAG = "${BUILD_NUMBER}"
        AWS_REGION = 'ap-south-1'
        EKS_CLUSTER_NAME = 'jenkins-eks-cluster'
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
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {

                    sh "sed -i 's|image:.*|image: ${IMAGE_NAME}:${TAG}|g' k8s-deployment.yaml"

                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}"

                    sh "kubectl apply -f k8s-deployment.yaml --validate=false"
                }
            }
        }

        stage('Verify Deployment Status') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {

                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}"

                    sh 'kubectl rollout status deployment/react-vite-app --timeout=120s'
                    sh 'kubectl get pods'
                    sh 'kubectl get svc'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }

        always {
            cleanWs()
        }
    }
}
