#!/usr/bin/env groovy
pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "us-east-1"
    }
    stages {
        stage("Create an EKS Cluster") {
            steps {
                script {
                    dir('terraform') {
                        sh "terraform init"
                        sh "terraform apply -auto-approve"
                    }
                }
            }
        }
        stage("Validate AWS Authentication") {
            steps {
                script {
                    // Check AWS authentication
                    sh "aws sts get-caller-identity"
                }
            }
        }
        stage("Deploy to EKS") {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                    ]) {
                        dir('kubernetes') {
                            // Update kubeconfig
                            sh "aws eks update-kubeconfig --name myapp-eks-cluster --region us-east-1"
                            
                            // Verify kubeconfig context
                            sh "kubectl config use-context arn:aws:eks:us-east-1:316065414151:cluster/myapp-eks-cluster"
                            
                            // Validate Kubernetes configuration
                            sh "kubectl get nodes"
                            
                            // Apply Kubernetes configuration
                            sh "kubectl apply -f jupiterapp.yaml"
                        }
                    }
                }
            }
        }
    }
    post {
        failure {
            script {
                echo 'Pipeline failed. Please check the logs for more details.'
            }
        }
        success {
            script {
                echo 'Pipeline completed successfully.'
            }
        }
    }
}


