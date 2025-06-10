pipeline {
    agent any

    environment {
        IMAGE_NAME = 'nginx-ec2'
        CONTAINER_NAME = 'nginx-server'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Stop Existing Container') {
            steps {
                sh 'docker stop $CONTAINER_NAME || true'
                sh 'docker rm $CONTAINER_NAME || true'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d --name $CONTAINER_NAME -p 80:80 $IMAGE_NAME'
            }
        }
    }
}

