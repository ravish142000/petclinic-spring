pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm        
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t springclinic:latest .'
            }
        }

        stage('Docker Login') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 103102677964.dkr.ecr.us-east-1.amazonaws.com'
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    docker tag springclinic:latest 103102677964.dkr.ecr.us-east-1.amazonaws.com/springclinic:latest 
                    docker push 103102677964.dkr.ecr.us-east-1.amazonaws.com/springclinic:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    kubectl set image deployment/customer-portal customer-portal=103102677964.dkr.ecr.us-east-1.amazonaws.com/springclinic:latest -n dev
                    kubectl rollout status deployment/customer-portal -n dev
                '''

            }
        }

    }
}
