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
      command: ["cat"]
      tty: true
      volumeMounts:
        - name: docker-socket
          mountPath: /var/run/docker.sock
    - name: helm-kubectl
      image: alpine/helm:3.13.2
      command: ["cat"]
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

    parameters {
        string(name: 'IMAGE', defaultValue: 'rangakrish/helix:latest', description: 'Docker image name')
        string(name: 'RELEASE', defaultValue: 'my-release', description: 'Helm release name')
        string(name: 'NAMESPACE', defaultValue: 'default', description: 'Kubernetes namespace')
        string(name: 'CHART_PATH', defaultValue: './my-helm-charts/helix-test/hello-world-chart', description: 'Path to Helm chart')
    }

    environment {
        KUBECONFIG = '/root/.kube/config'
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
                    withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                        docker build -t $IMAGE .
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $IMAGE
                        '''
                    }
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                container('helm-kubectl') {
                    sh '''
                    helm upgrade --install $RELEASE $CHART_PATH \
                      --set image.repository=${IMAGE%:*} \
                      --set image.tag=${IMAGE##*:} \
                      --namespace $NAMESPACE
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
            container('helm-kubectl') {
                archiveArtifacts artifacts: '**/*.log', fingerprint: true
            }
        }
        failure {
            container('helm-kubectl') {
                echo "Deployment failed! Rolling back Helm release..."
                sh '''
                helm rollback $RELEASE
                '''
            }
        }
        success {
            echo "Deployment successful!"
        }
    }
}
