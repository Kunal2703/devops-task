pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        ECR_REPO       = "205930634535.dkr.ecr.ap-south-1.amazonaws.com/devops-task"
        APP_NAME       = "devops-task"
        CLUSTER_NAME   = "demo-eks"      // replace with your EKS cluster name
        KUBE_NAMESPACE = "default"
        IMAGE_TAG      = "v${BUILD_NUMBER}"
    }

    stages {
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
                    docker build -t $APP_NAME:$IMAGE_TAG .

                    echo "Tagging Docker image with build version and latest..."
                    docker tag $APP_NAME:$IMAGE_TAG $ECR_REPO:$IMAGE_TAG
                    docker tag $APP_NAME:$IMAGE_TAG $ECR_REPO:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    echo "Pushing image to ECR..."
                    docker push $ECR_REPO:$IMAGE_TAG
                    docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    echo "Configuring kubectl for EKS..."
                    mkdir -p /var/lib/jenkins/.kube
                    aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME --kubeconfig /var/lib/jenkins/.kube/config
                    export KUBECONFIG=/var/lib/jenkins/.kube/config

                    echo "Updating deployment image..."
                    kubectl set image deployment/$APP_NAME $APP_NAME=$ECR_REPO:$IMAGE_TAG -n $KUBE_NAMESPACE

                    echo "Waiting for rollout..."
                    kubectl rollout status deployment/$APP_NAME -n $KUBE_NAMESPACE
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build $BUILD_NUMBER deployed successfully to $CLUSTER_NAME ($KUBE_NAMESPACE)"
        }
        failure {
            echo "❌ Build failed! Check logs."
        }
    }
}
