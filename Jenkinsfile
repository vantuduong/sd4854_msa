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
                        frontendApp.push("v1")
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
    }
}