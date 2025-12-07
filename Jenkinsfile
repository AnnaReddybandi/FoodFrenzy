pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'foodfrenzy'
        ECR_URI = "123456789012.dkr.ecr.ap-south-1.amazonaws.com/foodfrenzy"
        APP_JAR = 'FoodFrenzy-0.0.1-SNAPSHOT.jar'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning GitHub repository..."
                git 'https://github.com/AnnaReddybandi/FoodFrenzy.git'
            }
        }

        stage('Build JAR') {
            steps {
                echo "Building Spring Boot JAR..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image..."
                sh """
                    docker build -t $ECR_REPO:$IMAGE_TAG --build-arg JAR_FILE=target/$APP_JAR .
                """
            }
        }

        stage('Login to ECR') {
            steps {
                echo "Logging into AWS ECR..."
                sh """
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_URI
                """
            }
        }

        stage('Push to ECR') {
            steps {
                echo "Pushing Docker image to ECR..."
                sh """
                    docker tag $ECR_REPO:$IMAGE_TAG $ECR_URI:$IMAGE_TAG
                    docker push $ECR_URI:$IMAGE_TAG
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo "Deploying to EKS cluster..."
                sh """
                    kubectl set image deployment/foodfrenzy-deployment \
                    foodfrenzy=$ECR_URI:$IMAGE_TAG --record
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful! Image pushed as $IMAGE_TAG"
        }
        failure {
            echo "Build or deployment failed!"
        }
    }
}
