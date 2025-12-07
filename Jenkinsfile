pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'foodfrenzy'
        ECR_URI = "123456789012.dkr.ecr.ap-south-1.amazonaws.com/foodfrenzy"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/AnnaReddybandi/FoodFrenzy.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $ECR_REPO .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_URI
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                docker tag $ECR_REPO:latest $ECR_URI:latest
                docker push $ECR_URI:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}

