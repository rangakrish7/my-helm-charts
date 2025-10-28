pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: docker
      image: docker:20.10.24
      command:
        - cat
      tty: true
      volumeMounts:
        - name: docker-socket
          mountPath: /var/run/docker.sock
    - name: helm-kubectl
      image: alpine/helm:3.13.2
      command:
        - cat
      tty: true
      volumeMounts:
        - name: kube-config
          mountPath: /root/.kube
  volumes:
    - name: docker-socket
      hostPath:
        path: /var/run/docker.sock
    - name: kube-config
      hostPath:
        path: /root/.kube
"""
        }
    }
    environment {
        NAMESPACE = "default"
        RELEASE = "my-release"
        CHART_PATH = "./my-helm-charts/helix-test/hello-world-chart"
        IMAGE = "rangakrish/helix:latest"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                container('docker') {
                    sh '''
                    echo "Building Docker image..."
                    docker build -t $IMAGE .
                    docker login -u $DOCKER_USER -p $DOCKER_PASS
                    docker push $IMAGE
                    '''
                }
            }
        }
        stage('Deploy with Helm') {
            steps {
                container('helm-kubectl') {
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
        }
        stage('Capture Pod Logs') {
            steps {
                container('helm-kubectl') {
                    sh '''
                    POD_NAME=$(kubectl get pods --namespace $NAMESPACE -l app.kubernetes.io/instance=$RELEASE -o jsonpath="{.items[0].metadata.name}")
                    kubectl logs $POD_NAME --namespace $NAMESPACE > app-logs.log
                    '''
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '*.log', fingerprint: true
        }
    }
}
