pipeline {
    agent any

    environment {
        ECR_REPO = "992382545251.dkr.ecr.us-east-1.amazonaws.com/flask-jenkins-demo"
        IMAGE_TAG = "latest"
        AWS_REGION = "us-east-1"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/missviee/flask-jenkins-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ECR_REPO}:${IMAGE_TAG}", ".")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                // Use CLI instead of Jenkins docker object to avoid sandbox error
                sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
            }
        }

        stage('Deploy from ECR') {
            steps {
                sh "docker pull ${ECR_REPO}:${IMAGE_TAG}"

                // Stop and remove any existing container
                sh "docker stop flask-app || true"
                sh "docker rm flask-app || true"

                // Run the container
                sh "docker run -d --name flask-app -p 5000:5000 ${ECR_REPO}:${IMAGE_TAG}"
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }
    }
}

