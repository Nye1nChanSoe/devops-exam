pipeline {
    agent any

    environment {
        IMAGE_NAME = "ttl.sh/myapp-${env.BUILD_NUMBER}"
        IMAGE_TTL  = "2h"

        KUBE_SERVER = "https://kubernetes:6443"
        KUBE_TOKEN  = "kubernetes-secret"
    }

    stages {
        stage("Test") {
            agent {
                docker {
                    image 'node:24-alpine'
                    args '-u root'
                }
            }
            steps {
                sh '''
                    npm ci
                    node --test
                '''
            }
        }

        stage("Build & Push Image") {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:$IMAGE_TTL .
                    docker push $IMAGE_NAME:$IMAGE_TTL
                '''
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                withKubeConfig(
                    credentialsId: env.KUBE_TOKEN,
                    serverUrl: env.KUBE_SERVER
                ) {
                    sh '''
                        kubectl delete pod myapp --ignore-not-found

                        sed "s|image:.*|image: $IMAGE_NAME:$IMAGE_TTL|g" k8s/myapp.yml \
                          | kubectl apply -f -

                        kubectl wait --for=condition=Ready pod/myapp --timeout=120s

                        kubectl get pod myapp
                    '''
                }
            }
        }
    }
}