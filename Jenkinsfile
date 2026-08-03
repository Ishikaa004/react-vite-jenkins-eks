pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
            }
        }
        stage('Install & Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Dockerize & Deploy') {
            steps {
                echo 'Build successful! Ready for Docker & EKS deployment stages.'
            }
        }
    }
}
