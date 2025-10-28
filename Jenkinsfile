pipeline {
    agent any
    environment {
        KUBECONFIG = credentials('kubeconfig-cred')
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
                # Navigate to chart directory
                cd helix-test/hello-world-chart/helm-charts-main/charts/jenkins

                # Deploy using Helm
                helm upgrade --install my-app . \
                  --namespace default \
                  --values values.yaml
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
