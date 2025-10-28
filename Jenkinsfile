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
                helm repo add my-helm-charts https://rangakrish7.github.io/my-helm-charts/
                helm repo update

                # Deploy using Helm (replace chart name and namespace)
              
helm upgrade --install my-app my-helm-charts/hello-world-chart \
  --namespace default \
  --values values.yaml
''

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
