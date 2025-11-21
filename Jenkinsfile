pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')  // Jenkins credential ID
        DOCKERHUB_USER = "${DOCKERHUB_CREDENTIALS_USR}"
        DOCKERHUB_PASS = "${DOCKERHUB_CREDENTIALS_PSW}"
        IMAGE_NAME = "jaga0007/react-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    tools {
        nodejs "node20"
    }
    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('app') {
                    sh 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('app') {
                    sh 'npm test --if-present'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build \
                      -f docker/Dockerfile \
                      -t $IMAGE_NAME:$IMAGE_TAG \
                      -t $IMAGE_NAME:latest .
                """
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                sh """
                    trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_NAME:$IMAGE_TAG
                """
            }
            post {
                failure {
                    echo "❌ Security vulnerabilities found. Failing pipeline."
                }
            }
        }

        stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh """
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker push $IMAGE_NAME:latest
                """
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
