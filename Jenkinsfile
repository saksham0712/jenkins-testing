pipeline {
    agent any
    environment {
        ECR_REPO = "160885290762.dkr.ecr.ap-south-2.amazonaws.com/node-jenkins" // Replace with your ECR repo
        IMAGE_NAME = "node-fargate"
        AWS_CREDENTIALS_ID = "aws-credentials"
        GIT_REPO_URL = "https://github.com/saksham0712/jenkins-testing.git"  // URL of your Git repository
	BUILD_ID = "latest"
	GIT_BRANCH = "master"  
        GIT_CREDENTIALS_ID = "git-creds" 
    }
    stages {
        stage('Checkout Code') {
            steps {
                script {
                    git credentialsId: GIT_CREDENTIALS_ID, url: GIT_REPO_URL, branch: GIT_BRANCH
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${IMAGE_NAME}:${BUILD_ID} ."
                }
            }
        }
        stage('Login to ECR') {
            steps {
                script {
                    // Authenticate Docker with ECR
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                        sh """
                            aws ecr get-login-password --region ap-south-2 | docker login --username AWS --password-stdin 160885290762.dkr.ecr.ap-south-2.amazonaws.com
                        """
                    }
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    // Tag and push Docker image to ECR
                    sh "docker tag ${IMAGE_NAME}:${BUILD_ID} ${ECR_REPO}:${BUILD_ID}"
                    sh "docker push ${ECR_REPO}:${BUILD_ID}"
                }
            }
        }
    }
    post {
        always {
            // Clean up Docker images after build
            sh "docker rmi ${IMAGE_NAME}:${BUILD_ID}"
        }
    }
}
