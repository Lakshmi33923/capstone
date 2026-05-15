pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '364807861242.dkr.ecr.ap-south-1.amazonaws.com'

        IMAGE_NAME_NODE = 'node-app'
        IMAGE_NAME_SPRING = 'spring-app'
        IMAGE_NAME_FASTAPI = 'fastapi-app'

        EC2_INSTANCE_ID = 'i-081c91dfa42601c13'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Lakshmi33923/capstone-project2.git',
                    credentialsId: 'git-creds'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                set -e

                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                set -e

                # Spring Boot
                cd springapp/springapp
                docker build -t $IMAGE_NAME_SPRING .
                docker tag $IMAGE_NAME_SPRING:latest $ECR_REPO/$IMAGE_NAME_SPRING:latest
                cd ../../

                # Node
                cd nodeapp
                docker build -t $IMAGE_NAME_NODE .
                docker tag $IMAGE_NAME_NODE:latest $ECR_REPO/$IMAGE_NAME_NODE:latest
                cd ../

                # FastAPI
                cd fastapi_app
                docker build -t $IMAGE_NAME_FASTAPI .
                docker tag $IMAGE_NAME_FASTAPI:latest $ECR_REPO/$IMAGE_NAME_FASTAPI:latest
                cd ../
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                set -e

                docker push $ECR_REPO/$IMAGE_NAME_SPRING:latest
                docker push $ECR_REPO/$IMAGE_NAME_NODE:latest
                docker push $ECR_REPO/$IMAGE_NAME_FASTAPI:latest
                '''
            }
        }

        stage('Deploy to EC2 (SSM)') {
            steps {
                sh '''
                set -e

                aws ssm send-command \
                --instance-ids "$EC2_INSTANCE_ID" \
                --region "$AWS_REGION" \
                --document-name "AWS-RunShellScript" \
                --parameters 'commands=[
                    "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 364807861242.dkr.ecr.ap-south-1.amazonaws.com",

                    "docker pull 364807861242.dkr.ecr.ap-south-1.amazonaws.com/node-app:latest",
                    "docker pull 364807861242.dkr.ecr.ap-south-1.amazonaws.com/spring-app:latest",
                    "docker pull 364807861242.dkr.ecr.ap-south-1.amazonaws.com/fastapi-app:latest",

                    "docker stop nodeapp || true && docker rm nodeapp || true",
                    "docker stop springapp || true && docker rm springapp || true",
                    "docker stop fastapi || true && docker rm fastapi || true",

                    "docker run -d -p 3000:3000 --name nodeapp 364807861242.dkr.ecr.ap-south-1.amazonaws.com/node-app:latest",
                    "docker run -d -p 8080:8080 --name springapp 364807861242.dkr.ecr.ap-south-1.amazonaws.com/spring-app:latest",
                    "docker run -d -p 5000:5000 --name fastapi 364807861242.dkr.ecr.ap-south-1.amazonaws.com/fastapi-app:latest"
                ]'
                '''
            }
        }
    }
}
