pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        ECR_REPO       = "205930634535.dkr.ecr.ap-south-1.amazonaws.com/devops-task"
        APP_NAME       = "devops-task"
        CLUSTER_NAME   = "demo-eks"
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
                    echo "Skipping local build & test — handled inside Dockerfile"
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
                    echo "Configuring kubectl for EKS..."
                    aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME

                    echo "Navigating to workspace..."
                    cd $WORKSPACE

                    echo "Applying Kubernetes manifests..."
                    kubectl apply -f deployment.yaml -n $KUBE_NAMESPACE
                    kubectl apply -f service.yaml -n $KUBE_NAMESPACE

                    echo "Waiting for rollout..."
                    kubectl rollout status deployment/$APP_NAME -n $KUBE_NAMESPACE
                '''
            }
        }
    }
}
