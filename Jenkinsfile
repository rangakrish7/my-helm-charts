pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17' // Maven + JDK
            args '-v /root/.m2:/root/.m2' // Cache Maven dependencies
        }
    }
    environment {
        KUBECONFIG = credentials('kubeconfig-id') // Jenkins credential for Kubernetes cluster
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
                # Add Helm repo
                helm repo add my-helm-charts https://github.com/rangakrish7/my-helm-charts.git
                helm repo update

                # Deploy using Helm
                helm upgrade --install my-app my-helm-charts/my-chart \
                  --namespace my-namespace \
                  --values values-${env.BRANCH_NAME}.yaml
                '''
            }
        }
    }
    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
