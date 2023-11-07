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
        stage ("Trivy Scan Secret") {
            steps {
                script {
                    sh "trivy fs --severity HIGH,CRITICAL --format template --template @trivy/html.tpl -o trivy/secretreport.html ."
                    publishHTML (target : [allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'trivy',
                        reportFiles: 'secretreport.html',
                        reportName: 'Trivy Secrets Report',
                        reportTitles: 'Trivy Secrets Report']
                    )
                }
            }
        }

        stage('Build frontend') {
            steps {
                script{
                    frontendApp = docker.build("devops-jenkins-frontend", "-f src/frontend/Dockerfile src/frontend")
                    sh "docker images"
                }
            }
        }

        stage ("Trivy Scan Frontend Images") {
            steps {
                script {
                    sh "trivy image --scanners vuln,config --exit-code 0 --severity HIGH,CRITICAL --format template --template @trivy/html.tpl -o trivy/frontendImageReport.html devops-jenkins-frontend:latest"
                    publishHTML (target : [allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'trivy',
                        reportFiles: 'frontendImageReport.html',
                        reportName: 'Trivy Vulnerabilities Frontend Images Report',
                        reportTitles: 'Trivy Vulnerabilities Frontend Images Report']
                    )
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

         stage ("Trivy Scan Backend Images") {
             steps {
                script {
                    sh "trivy image --scanners vuln,config --exit-code 0 --severity HIGH,CRITICAL --format template --template @trivy/html.tpl -o trivy/backendImageReport.html devops-jenkins-backend:latest"
                    publishHTML (target : [allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'trivy',
                        reportFiles: 'backendImageReport.html',
                        reportName: 'Trivy Vulnerabilities Backend Images Report',
                        reportTitles: 'Trivy Vulnerabilities Backend Images Report']
                    )
                }
             }
        }

        stage('Push frontend image'){
            steps {
                script{
                    docker.withRegistry('https://258591199682.dkr.ecr.ap-southeast-1.amazonaws.com', 'ecr:ap-southeast-1:aws-credentials') {
                        frontendApp.push("green")
                    }
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