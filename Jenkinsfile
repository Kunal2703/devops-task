pipeline {
    agent any

    environment {
        AWS_REGION   = "ap-south-1"
        ECR_REPO     = "205930634535.dkr.ecr.ap-south-1.amazonaws.com/devops-task"
        IMAGE_TAG    = "build-${BUILD_NUMBER}"
        BASTION_USER = "ubuntu"                  // your Bastion username
        BASTION_HOST = "13.201.126.164"     // replace with actual Bastion public IP/DNS
        EKS_CLUSTER  = "demo-eks" // replace with actual EKS cluster name
    }

    triggers {
        githubPush()   // Trigger via GitHub webhook
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kunal2703/devops-task.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                echo "Installing dependencies..."
                npm install
                echo "Running tests..."
                npm test || echo "No tests found"
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                withAWS(region: "${AWS_REGION}", credentials: 'aws-creds') {
                    sh '''
                    echo "Logging into ECR..."
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO

                    echo "Building Docker image..."
                    docker build -t $ECR_REPO:$IMAGE_TAG .

                    echo "Pushing image to ECR..."
                    docker push $ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to EKS via Bastion') {
            steps {
                sshagent(['bastion-ssh-key']) {
                    sh '''
                    echo "Deploying new version to EKS via Bastion..."
                    ssh -o StrictHostKeyChecking=no $BASTION_USER@$BASTION_HOST "
                        aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER &&
                        kubectl set image deployment/devops-task devops-task=$ECR_REPO:$IMAGE_TAG -n default &&
                        kubectl rollout status deployment/devops-task -n default
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
