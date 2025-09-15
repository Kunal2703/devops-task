pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "205930634535.dkr.ecr.ap-south-1.amazonaws.com/devops-task"
        APP_NAME = "devops-task"
        CLUSTER_NAME = "demo-eks"   // update with your cluster name
        KUBE_NAMESPACE = "default"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Kunal2703/devops-task.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    echo "Installing dependencies & running tests"
                    npm install
                    npm test
                '''
            }
        }

        stage('Dockerize') {
            steps {
                sh '''
                    echo "Logging into ECR..."
                    aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REPO

                    echo "Building Docker image..."
                    docker build -t $APP_NAME:latest .

                    echo "Tagging Docker image..."
                    docker tag $APP_NAME:latest $ECR_REPO:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    echo "Pushing image to ECR..."
                    docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    echo "Deploying to EKS..."

                    kubectl create deployment $APP_NAME --image=$ECR_REPO:latest -n $KUBE_NAMESPACE || \
                    kubectl set image deployment/$APP_NAME $APP_NAME=$ECR_REPO:latest -n $KUBE_NAMESPACE

                    kubectl rollout status deployment/$APP_NAME -n $KUBE_NAMESPACE
                '''
            }
        }
    }
}
