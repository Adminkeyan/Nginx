pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        REPO_NAME = 'nginx'
        AWS_ACCOUNT_ID = '489502444480' // Replace with your AWS Account ID
        IMAGE_TAG = 'latest'
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
        EC2_HOST = 'ubuntu@13.232.249.252' // Change if AMI uses ec2-user
        SSH_KEY = credentials('ec2-key') // Jenkins secret with PEM key content
        AWS_ACCESS_KEY_ID = credentials('aws-access-key') // Add AWS_ACCESS_KEY_ID as a secret
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key') // Add AWS_SECRET_ACCESS_KEY as a secret
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REPO_NAME .'
            }
        }

        stage('Login to AWS & ECR') {
            steps {
                sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region $AWS_REGION

                    aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_URL
                '''
            }
        }

        stage('Tag & Push Docker Image to ECR') {
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
                    echo "$SSH_KEY" > ec2-key.pem
                    chmod 400 ec2-key.pem

                    ssh -o StrictHostKeyChecking=no -i ec2-key.pem $EC2_HOST <<EOF
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region $AWS_REGION

                        aws ecr get-login-password --region $AWS_REGION | \
                            docker login --username AWS --password-stdin $ECR_URL

                        docker pull $ECR_URL:$IMAGE_TAG
                        docker stop nginx || true
                        docker rm nginx || true
                        docker run -d --name nginx -p 80:80 $ECR_URL:$IMAGE_TAG
                    EOF
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -f ec2-key.pem' // Clean up SSH key after job
        }
    }
}
