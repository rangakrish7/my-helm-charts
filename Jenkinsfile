pipeline {
    agent any
    environment {
        KUBECONFIG = credentials('kubeconfig-cred') // Jenkins credential for cluster
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package' // or your build command
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test' // or your test command
            }
        }
        stage('Deploy with Helm') {
            steps {
                sh '''
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
}
