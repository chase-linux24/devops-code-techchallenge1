pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-2'
        AWS_ACCOUNT_ID = '381491979307'
        ECR_FRONTEND = "381491979307.dkr.ecr.us-east-2.amazonaws.com/devops-challenge-frontend"
        ECR_BACKEND = "381491979307.dkr.ecr.us-east-2.amazonaws.com/devops-challenge-backend"
        ECS_CLUSTER = 'devops-challenge-cluster'
        ECS_FRONTEND_SERVICE = 'devops-challenge-frontend-service'
        ECS_BACKEND_SERVICE = 'devops-challenge-backend-service'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scm
            }
        }

        stage('Login to ECR') {
            steps {
                echo 'Logging into Amazon ECR...'
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building frontend and backend Docker images...'
                sh '''
                    docker build -t ${ECR_FRONTEND}:${IMAGE_TAG} ./frontend
                    docker build -t ${ECR_BACKEND}:${IMAGE_TAG} ./backend
                '''
            }
        }

        stage('Push to ECR') {
    steps {
        echo 'Pushing images to ECR...'
        sh '''
            docker push ${ECR_FRONTEND}:${IMAGE_TAG}
            docker push ${ECR_BACKEND}:${IMAGE_TAG}
            
            docker tag ${ECR_FRONTEND}:${IMAGE_TAG} ${ECR_FRONTEND}:latest
            docker tag ${ECR_BACKEND}:${IMAGE_TAG} ${ECR_BACKEND}:latest
            
            docker push ${ECR_FRONTEND}:latest
            docker push ${ECR_BACKEND}:latest
        '''
    }
}

        stage('Deploy to ECS') {
            steps {
                echo 'Deploying to ECS Fargate...'
                sh '''
                    aws ecs update-service \
                        --cluster ${ECS_CLUSTER} \
                        --service ${ECS_FRONTEND_SERVICE} \
                        --force-new-deployment \
                        --region ${AWS_REGION}

                    aws ecs update-service \
                        --cluster ${ECS_CLUSTER} \
                        --service ${ECS_BACKEND_SERVICE} \
                        --force-new-deployment \
                        --region ${AWS_REGION}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying ECS services are stable...'
                sh '''
                    aws ecs wait services-stable \
                        --cluster ${ECS_CLUSTER} \
                        --services ${ECS_FRONTEND_SERVICE} \
                        --region ${AWS_REGION}

                    aws ecs wait services-stable \
                        --cluster ${ECS_CLUSTER} \
                        --services ${ECS_BACKEND_SERVICE} \
                        --region ${AWS_REGION}
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully! Application deployed to ECS.'
        }
        failure {
            echo 'Pipeline failed. Check the logs above for errors.'
        }
    }
}