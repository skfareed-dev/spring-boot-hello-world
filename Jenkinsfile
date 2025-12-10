pipeline {
    agent any
    options {
        skipDefaultCheckout()
    }

    environment {
        AWS_REGION = "ap-southeast-1"
        ECR_REPO = "369138027325.dkr.ecr.ap-southeast-1.amazonaws.com/my-java-app"
        IMAGE_TAG = "latest"
        DEPLOY_SERVER = "10.0.20.156"
    }

    tools {
        maven 'Maven3'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/skfareed-dev/spring-boot-hello-world.git'
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

        stage('Tag & Push to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh '''
                        # ECR Login
                        aws ecr get-login-password --region ${AWS_REGION} \
                        | docker login --username AWS --password-stdin ${ECR_REPO}

                        # Tag Image
                        docker tag my-java-app:latest ${ECR_REPO}:${IMAGE_TAG}

                        # Push Image to ECR
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EC2 via SSH') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOY_SERVER} '
                        cd ~/deploy &&
                        docker compose pull &&
                        docker compose down &&
                        docker compose up -d
                    '
                    """
                }
            }
        }
        post {
    success {
        emailext (
            subject: "SUCCESS: Build ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
Hello Team,
The Jenkins build has completed successfully.
Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Status: SUCCESS

Regards,
devops team
""",
            to: "your@email.com"
        )
    }

    failure {
        emailext (
            subject: "FAILED: Build ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
Hello Team,
The Jenkins build has FAILED.
Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Status: FAILURE

Check console output: ${env.BUILD_URL}console

Regards,
devops team
""",
            to: "your@email.com"
        )
    }
}

    }
}
