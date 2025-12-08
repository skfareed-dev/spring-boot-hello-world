pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    environment {
        AWS_REGION = "ap-southeast-1"
        ECR_REPO = "369138027325.dkr.ecr.ap-southeast-1.amazonaws.com/my-java-app"
        IMAGE_TAG = "latest"
    }

    tools {
        maven 'Maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm
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