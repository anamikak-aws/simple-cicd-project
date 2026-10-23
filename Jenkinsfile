pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t simple-cicd-website .'
            }
        }

        stage('Deploy Website') {
            steps {
                sh '''
                    docker stop simple-cicd-container || true
                    docker rm simple-cicd-container || true
                    docker run -d --name simple-cicd-container -p 8081:80 simple-cicd-website
                '''
            }
        }
    }
}
