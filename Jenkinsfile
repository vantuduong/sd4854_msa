pipeline {
    agent any
    environment {
        REGISTRY_URL = 'https://720766170633.dkr.ecr.us-east-2.amazonaws.com'
        EKS_NAMESPACE = 'eks-ns'
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
                    frontendApp = docker.build("frontend", "-f src/frontend/Dockerfile .")
                }
            }
        }
        stage('Push frontend image'){
            steps {
                script{
                    docker.withRegistry('$REGISTRY_URL', 'ecr:us-east-2:aws-credentials') {
                        frontendApp.push("${env.BUILD_NUMBER}")
                        frontendApp.push("latest")
                    }
                }
            }
        }
        stage('Build backend') {
            steps {
                script{
                    backendApp = docker.build("backend", "-f src/backend/Dockerfile .")
                }
            }
        }
        stage('Push frontend image'){
            steps {
                script{
                    docker.withRegistry('$REGISTRY_URL', 'ecr:us-east-2:aws-credentials') {
                        backendApp.push("${env.BUILD_NUMBER}")
                        backendApp.push("latest")
                    }
                }
            }
        }
        stage('Set default namespace'){
            steps {
                 sh 'kubectl config set-context --current --namespace $EKS_NAMESPACE'
            }
        }

        stage('Deploy mongo db'){
            steps {
                 sh 'kubectl apply -f mongodb.yaml'
                 sh 'kubectl get pod'
                 sh 'kubectl get service'
            }
        }

        stage('Deploy backend'){
            steps {
                 sh 'kubectl apply -f backend.yaml'
                 sh 'kubectl get pod'
                 sh 'kubectl get service'
            }
        }

        stage('Deploy frontend'){
            steps {
                 sh 'kubectl apply -f frontend.yaml'
                 sh 'kubectl get pod'
                 sh 'kubectl get service'
                 sh 'kubectl port-forward service/frontend 3000:3000'
            }
        }

        stage('Deploy nginx'){
            steps {
                 sh 'kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/cloud/deploy.yaml'
                 sh 'kubectl get pod --namespace=ingress-nginx | grep nginx'
                 sh 'kubectl apply -f ingress.yml'
                 sh 'kubectl get ingress -o wide'
            }
        }
    }
}