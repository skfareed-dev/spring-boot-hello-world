pipeline {
    agent any

    environment {
        AWS_REGION = "ap-southeast-1"
        ECR_REPO = "369138027325.dkr.ecr.ap-southeast-1.amazonaws.com/my-java-app"
        IMAGE_TAG = "latest"
    }

    tools {
        maven 'Maven3'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/skfareed-dev/spring-boot-hello-world.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t my-java-app .'
            }
        }

        stage('Tag Image') {
            steps {
                sh "docker tag my-java-app:latest ${ECR_REPO}:${IMAGE_TAG}"
            }
        }

        stage('ECR Login') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
            }
        }
    }
}
