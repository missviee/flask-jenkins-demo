pipeline {
    agent any

    environment {
        ECR_REPO = "992382545251.dkr.ecr.us-east-1.amazonaws.com/flask-jenkins-demo"
        IMAGE_TAG = "latest"
        AWS_REGION = "us-east-1"

        EC2_USER = "ubuntu"
        EC2_HOST = "44.200.75.94"  // new public IP
        SSH_CREDENTIALS = "ec2-ssh-credentials-id"
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
                script {
                    docker.push("${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent([SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                        docker pull ${ECR_REPO}:${IMAGE_TAG} &&
                        docker stop flask-app || true &&
                        docker rm flask-app || true &&
                        docker run -d -p 5000:5000 --name flask-app ${ECR_REPO}:${IMAGE_TAG}'
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }
    }
}

