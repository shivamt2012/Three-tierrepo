pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-2"
        ECR_REPO = "590183997695.dkr.ecr.us-east-2.amazonaws.com/threetier-demo"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t threetier-demo .'
            }
        }

        stage('Authenticate to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin 590183997695.dkr.ecr.us-east-2.amazonaws.com
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                docker tag threetier-demo:latest $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

       stage('Deploy to Target Server') {
    steps {
        sshagent(['server-ssh-key']) {
            sh """
            ssh -o StrictHostKeyChecking=no ubuntu@54.155.64.128 '
                aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 590183997695.dkr.ecr.us-east-2.amazonaws.com &&
                docker pull 590183997695.dkr.ecr.us-east-2.amazonaws.com/threetier-demo:latest &&
                docker stop app || true &&
                docker rm app || true &&
                docker run -d -p 3000:3000 --name app 590183997695.dkr.ecr.us-east-2.amazonaws.com/threetier-demo:latest
            '
            """
        }
    }
}
