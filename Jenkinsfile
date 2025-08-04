pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID = '619071339639'
        AWS_REGION = 'eu-west-1'
        IMAGE_REPO_NAME = 'jenkins-app-repo'
        IMAGE_TAG = 'latest'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Msocial123/jenkins-app.git'
            }
        }
        stage('Build Image') {
            steps {
                sh 'docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .'
            }
        }
        stage('Login to ECR') {
            steps {
                sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com'
            }
        }
        stage('Tag and Push to ECR') {
            steps {
                sh 'docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG'
                sh 'docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG'
            }
        }
        stage('Scan Image with Trivy') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity HIGH,CRITICAL $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG'
            }
        }
        stage('Deploy Application') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }
}
