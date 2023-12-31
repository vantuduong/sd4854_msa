pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Clone repository') {
            steps {
                script{
                    checkout scm
                }
            }
        }
        stage('Build frontend') {
            steps {
                script{
                    frontendApp = docker.build("devops-jenkins-frontend", "-f src/frontend/Dockerfile src/frontend")
                }
            }
        }
        stage('Push frontend image'){
            steps {
                script{
                    docker.withRegistry('https://258591199682.dkr.ecr.ap-southeast-1.amazonaws.com', 'ecr:ap-southeast-1:aws-credentials') {
                        frontendApp.push("latest")
                    }
                }
            }
        }
        stage('Build backend') {
            steps {
                script{
                    backendApp = docker.build("devops-jenkins-backend", "-f src/backend/Dockerfile src/backend")
                }
            }
        }
        stage('Push backend image'){
            steps {
                script{
                    docker.withRegistry('https://258591199682.dkr.ecr.ap-southeast-1.amazonaws.com', 'ecr:ap-southeast-1:aws-credentials') {
                        backendApp.push("latest")
                    }
                }
            }
        }

        stage('Deploy backend'){
            steps {
                script {
                    sh 'kubectl apply -f backend.yaml'
                    sh 'kubectl get pod'
                    sh 'kubectl get service'
                }
            }
        }

        stage('Deploy frontend'){
            steps {
                script {
                    sh 'kubectl apply -f frontend.yaml'
                    sh 'kubectl get pod'
                    sh 'kubectl get service'
                }
            }
        }

        stage('Deploy mongo db'){
            steps {
                script {
                    sh 'kubectl apply -f mongodb.yaml'
                    sh 'kubectl get pod'
                    sh 'kubectl get service'
                }
            }
        }

        stage('Deploy nginx'){
            steps {
                 script {
                    sh 'kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/cloud/deploy.yaml'
                    sh 'kubectl apply -f ingress.yml'
                 }
            }
        }
    }
}