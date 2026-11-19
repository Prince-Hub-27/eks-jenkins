
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-southeast-2'
        AWS_ACCOUNT_ID = '712685518638'
        EKS_CLUSTER = 'eks-jenkins-cluster'
        DEPLOYMENT_NAME = 'dotnet-api'
        CONTAINER_NAME = 'dotnet-api'
        IMAGE_URI = '712685518638.dkr.ecr.ap-southeast-2.amazonaws.com/dotnet-api'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_URI:$BUILD_NUMBER .
                '''
            }
        }

        stage('Push to Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION |
                    docker login --username AWS --password-stdin \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                    docker push $IMAGE_URI:$BUILD_NUMBER
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    mkdir -p "$WORKSPACE/.kube"

                    aws eks update-kubeconfig \
                        --region $AWS_REGION \
                        --name $EKS_CLUSTER \
                        --kubeconfig "$WORKSPACE/.kube/config"

                    export KUBECONFIG="$WORKSPACE/.kube/config"

                    # Apply Kubernetes configuration
                    kubectl apply -f deployment.yaml

                    # Update the application image
                    kubectl set image deployment/$DEPLOYMENT_NAME \
                        $CONTAINER_NAME=$IMAGE_URI:$BUILD_NUMBER

                    # Wait for deployment
                    kubectl rollout status deployment/$DEPLOYMENT_NAME \
                        --timeout=180s

                    kubectl get pods -l app=dotnet-api
                '''
            }
        }
    }
}
