pipeline {
    agent any
    environment {
        REGISTRY_URL = 'https://258591199682.dkr.ecr.ap-southeast-1.amazonaws.com'
    }
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
                    frontendApp = docker.build("devops-jenkins/frontend", "-f src/frontend/Dockerfile src/frontend")
                }
            }
        }
        stage('Push frontend image'){
            steps {
                script{
                    docker.withRegistry('$REGISTRY_URL', 'ecr:ap-southeast-1:aws-credentials') {
                        frontendApp.push("${env.BUILD_NUMBER}")
                        frontendApp.push("latest")
                    }
                }
            }
        }
        stage('Build backend') {
            steps {
                script{
                    backendApp = docker.build("devops-jenkins/backend", "-f src/backend/Dockerfile src/backend")
                }
            }
        }
        stage('Push backend image'){
            steps {
                script{
                    docker.withRegistry('$REGISTRY_URL', 'ecr:ap-southeast-1:aws-credentials') {
                        backendApp.push("${env.BUILD_NUMBER}")
                        backendApp.push("latest")
                    }
                }
            }
        }

        stage('Deploy mongo db'){
            steps {
                script{
                    kubeconfig(credentialsId: 'kubernetesCredential', serverUrl: '') {
                        sh 'kubectl apply -f mongodb.yaml'
                        sh 'kubectl get pod'
                        sh 'kubectl get service'
                    }
                }
            }
        }

        stage('Deploy backend'){
            steps {
                script{
                    kubeconfig(credentialsId: 'kubernetesCredential', serverUrl: '') {
                        sh 'kubectl apply -f backend.yaml'
                        sh 'kubectl get pod'
                        sh 'kubectl get service'
                    }
                }
            }
        }

        stage('Deploy frontend'){
            steps {
                script{
                    kubeconfig(credentialsId: 'kubernetesCredential', serverUrl: '') {
                        sh 'kubectl apply -f frontend.yaml'
                        sh 'kubectl get pod'
                        sh 'kubectl get service'
                        sh 'kubectl port-forward service/frontend 3000:3000'
                    }
                }
            }

        }
    }
}