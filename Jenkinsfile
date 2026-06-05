pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REGISTRY = "811822680488.dkr.ecr.us-east-1.amazonaws.com"
        IMAGE_NAME = "springclinic"
        ECR_REPO = "${ECR_REGISTRY}/${IMAGE_NAME}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh "./mvnw clean package"
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
            }
        }

        stage('Docker Login') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Docker Tag & Push') {
            steps {
                sh """
                    docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${ECR_REPO}:${env.BUILD_NUMBER}
                    docker push ${ECR_REPO}:${env.BUILD_NUMBER}
                """
            }
        }

        stage('Deploy to DEV') {
            steps {
                sh """
                    kubectl set image deployment/customer-portal \
                    customer-portal=${ECR_REPO}:${env.BUILD_NUMBER} \
                    -n dev

                    kubectl rollout status deployment/customer-portal -n dev
                """
            }
        }

        stage('Approval for PROD') {
            steps {
                input message: "Deploy build ${env.BUILD_NUMBER} to production?"
            }
        }

        stage('Deploy to PROD') {
            steps {
                sh """
                    kubectl set image deployment/customer-portal \
                    customer-portal=${ECR_REPO}:${env.BUILD_NUMBER} \
                    -n prod

                    kubectl rollout status deployment/customer-portal -n prod
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully. Build: ${env.BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
