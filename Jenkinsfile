pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        REPO_NAME = 'nginx'
        AWS_ACCOUNT_ID = '489502444480'  // replace with yours
        IMAGE_TAG = 'latest'
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
        EC2_HOST = 'ubuntu@<13.232.249.252>'  // use Ubuntu or ec2-user
        SSH_KEY = credentials('ec2-key')       // Add SSH private key in Jenkins
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Adminkeyan/Nginx.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REPO_NAME .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                aws configure set region $AWS_REGION
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URL
                '''
            }
        }

        stage('Tag & Push Image to ECR') {
            steps {
                sh '''
                docker tag $REPO_NAME:latest $ECR_URL:$IMAGE_TAG
                docker push $ECR_URL:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no -i $SSH_KEY $EC2_HOST << 'EOF'
                  aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URL
                  docker pull $ECR_URL:$IMAGE_TAG
                  docker stop nginx || true
                  docker rm nginx || true
                  docker run -d --name nginx -p 80:80 $ECR_URL:$IMAGE_TAG
                EOF
                '''
            }
        }
    }
}
