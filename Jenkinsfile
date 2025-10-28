pipeline {
    agent any
    environment {
        KUBECONFIG = credentials('kubeconfig-cred') // Jenkins credential for Kubernetes cluster
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Deploy with Helm') {
            steps {
                sh '''
                # Add Helm repo (replace with your actual chart repo URL)
                helm repo add my-helm-charts https://github.com/rangakrish7/my-helm-charts.git
                helm repo update

                # Deploy using Helm (replace chart name and namespace)
                helm upgrade --install my-app my-helm-charts/my-chart \
                  --namespace my-namespace \
                  --values values-${env.BRANCH_NAME}.yaml
                '''
            }
        }
    }
    post {
        success {
            echo "✅ Helm deployment successful!"
        }
        failure {
            echo "❌ Helm deployment failed!"
        }
    }
}
