pipeline {
    agent any

    environment {
        // Set your ECR repository details here
        // The ECR_URI is the full URI you provided
        ECR_URI = '619071339639.dkr.ecr.eu-west-1.amazonaws.com/jevan-ecr-repo'
        // The AWS_REGION for ECR
        AWS_REGION = 'eu-west-1'
        // Use a consistent image tag, e.g., using the build number
        IMAGE_NAME = "jevan-app:${BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                // The Git clone is handled automatically by the Jenkins job configuration
                echo "Cloning repository..."
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${IMAGE_NAME} .'
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Authenticate Docker with ECR using the AWS CLI
                    sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}'
                    // Tag the image for ECR
                    sh 'docker tag ${IMAGE_NAME} ${ECR_URI}:${IMAGE_NAME}'
                    // Push the image to ECR
                    sh 'docker push ${ECR_URI}:${IMAGE_NAME}'
                }
            }
        }

        stage('Scan Docker Image with Trivy') {
            steps {
                script {
                    // Install Trivy if it's not already installed
                    sh 'sudo dnf install trivy -y || true'
                    // Scan the image for vulnerabilities
                    sh 'trivy --exit-code 0 --format table --vuln-type os,library --severity CRITICAL,HIGH,MEDIUM,LOW ${ECR_URI}:${IMAGE_NAME}'
                }
            }
        }
        
        stage('Deploy with Docker Compose') {
            steps {
                script {
                    // Pull the newly built image from ECR
                    sh 'docker-compose pull'
                    // Start the container using docker-compose
                    sh 'docker-compose up -d'
                }
            }
        }
    }
}
