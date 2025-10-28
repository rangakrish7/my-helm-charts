pipeline {
    agent any
    environment {
        NAMESPACE = "default"
        RELEASE = "my-release"
        CHART_PATH = "./my-helm-charts/helix-test/hello-world-chart"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image..."
                docker build -t rangakrish/helix:latest .
                docker login -u $DOCKER_USER -p $DOCKER_PASS
                docker push rangakrish/helix:latest
                '''
            }
        }
        stage('Deploy with Helm') {
            steps {
                sh '''
                echo "Deploying Helm release..."
                helm upgrade --install $RELEASE $CHART_PATH --namespace $NAMESPACE
                helm history $RELEASE --namespace $NAMESPACE > helm-history.log
                helm get values $RELEASE --namespace $NAMESPACE > helm-values.log
                kubectl get pods --namespace $NAMESPACE > pods.log
                kubectl describe deployment $RELEASE-hello-world-chart --namespace $NAMESPACE > deployment-details.log
                '''
            }
        }
        stage('Capture Pod Logs') {
            steps {
                sh '''
                POD_NAME=$(kubectl get pods --namespace $NAMESPACE -l app.kubernetes.io/instance=$RELEASE -o jsonpath="{.items[0].metadata.name}")
                kubectl logs $POD_NAME --namespace $NAMESPACE > app-logs.log
                '''
            }
        }
    }
    post {
        always {
            echo "Archiving logs for traceability..."
            archiveArtifacts artifacts: '*.log', fingerprint: true
        }
    }
}
