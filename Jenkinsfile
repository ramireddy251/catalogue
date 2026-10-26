pipeline {
    agent {
        node {
            label 'Roboshop'
        }
    }
    environment {
        appVersion = ""
        AWS_ACCOUNT_ID = "104050870679"
        AWS_REGION     = "us-east-1"
        ECR_REPO_NAME  = "roboshop/catalogue"
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }
    options {
        timeout(time: 5, unit: 'MINUTES')
    } 
    stages {
        stage('Read version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Building version ${appVersion}"
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Build Image') {
            steps {
                sh "docker build -t ${ECR_REPO_NAME}:${appVersion} ."
            }
        }
        stage('Push to ECR') {
            steps {
                // Securely authenticate, tag, and push using AWS CLI and Docker
                sh """
                # 1. Authenticate Docker daemon to ECR registry
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                
                # 2. Tag the local image with the ECR repository URL
                docker tag ${ECR_REPO_NAME}:${appVersion} ${ECR_REGISTRY}/${ECR_REPO_NAME}:${appVersion}
                docker tag ${ECR_REPO_NAME}:${appVersion} ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest
                
                # 3. Push both the versioned tag and 'latest' tag to ECR
                docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:${appVersion}
                docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest
                """
            }
        }
    }

    post {
        always {
            echo "I will always say Hello again!"
        }
        success {
            echo "pipeline success"
        }
        failure {
            echo "pipeline failure"
        }
    }
}
