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

    environment {
        IMAGE = "rangakrish/helix:latest"
        RELEASE = "my-release"
        NAMESPACE = "default"
        CHART_PATH = "./my-helm-charts/helix-test/hello-world-chart"
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

        stage('Archive Logs') {
            steps {
                archiveArtifacts artifacts: '**/*.log', fingerprint: true
            }
        }

        stage('Rollback on Failure') {
            when {
                expression { currentBuild.result == 'FAILURE' }
            }
            steps {
                container('helm-kubectl') {
                    sh "helm rollback $RELEASE"
                }
            }
        }
    }
}
