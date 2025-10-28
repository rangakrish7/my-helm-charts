pipeline {
    agent any
    tools {
        maven 'Maven-3.9.6' // Configure in Jenkins Global Tool Configuration
    }
    environment {
        KUBECONFIG = credentials('kubeconfig-cred')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy with Helm') {
            steps {
                sh '''
                helm repo add my-helm-charts https://charts.example.com
                helm repo update
                helm upgrade --install my-app my-helm-charts/my-chart \
                  --namespace my-namespace \
                  --values values-${env.BRANCH_NAME}.yaml
                '''
            }
        }
    }
}
