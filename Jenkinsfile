
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'simple-cicd-website'
        CONTAINER_NAME = 'simple-cicd-container'
        APP_PORT = '8081'
    }

    stages {
        stage('1. Checkout') {
            steps {
                checkout scm
            }
        }

        stage('2. Verify Files') {
            steps {
                sh 'test -f index.html && test -f Dockerfile'
                sh 'ls -la'
            }
        }

        stage('3. Validate HTML') {
            steps {
                sh 'test -s index.html'
                sh 'grep -qi "<html" index.html'
            }
        }

        stage('4. Validate Dockerfile') {
            steps {
                sh 'test -s Dockerfile'
                sh 'cat Dockerfile'
            }
        }

        stage('5. Check Docker') {
            steps {
                sh 'docker --version'
                sh 'docker info'
            }
        }

        stage('6. Build Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('7. Verify Image') {
            steps {
                sh 'docker image inspect ${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }

        stage('8. Test Container') {
            steps {
                sh '''
                    docker rm -f cicd-test-container 2>/dev/null || true
                    docker run -d --name cicd-test-container \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                    sleep 3
                    docker inspect -f '{{.State.Running}}' cicd-test-container \
                        | grep -x true
                    docker rm -f cicd-test-container
                '''
            }
        }

        stage('9. Deploy Website') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${APP_PORT}:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('10. HTTP Health Check') {
            steps {
                sh '''
                    curl --retry 5 --retry-delay 2 --retry-connrefused \
                        -fsS http://localhost:${APP_PORT}/ -o /dev/null
                    echo "HTTP health check passed"
                '''
            }
        }

        stage('11. Show Container Status') {
            steps {
                sh 'docker ps --filter name=${CONTAINER_NAME}'
                sh 'docker logs --tail 20 ${CONTAINER_NAME}'
            }
        }

        stage('12. Deployment Summary') {
            steps {
                sh '''
                    echo "Deployment successful"
                    echo "Build: ${BUILD_NUMBER}"
                    echo "Image: ${IMAGE_NAME}:${BUILD_NUMBER}"
                    echo "Website port: ${APP_PORT}"
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'

            emailext(
                to: 'YOUR_EMAIL@example.com',
                subject: "SUCCESS: ${JOB_NAME} - Build #${BUILD_NUMBER}",
                body: """Hello,

Your CI/CD pipeline completed successfully.

Job: ${JOB_NAME}
Build: #${BUILD_NUMBER}
Status: ${currentBuild.currentResult}

Build details:
${BUILD_URL}
"""
            )
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'

            emailext(
                to: 'anamika.k.aws@gmail.com',
                subject: "FAILED: ${JOB_NAME} - Build #${BUILD_NUMBER}",
                body: """Hello,

Your CI/CD pipeline has failed.

Job: ${JOB_NAME}
Build: #${BUILD_NUMBER}
Status: ${currentBuild.currentResult}

Check the Console Output:
${BUILD_URL}console
"""
            )
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}

