pipeline {
    agent any

    environment {
        // with build number
        IMAGE_NAME = "ttl.sh/myapp-${env.BUILD_NUMBER}"
        IMAGE_TTL  = "2h"
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

        stage("Deploy") {
            steps {
                sshagent(credentials: ['secret-key']) {
                    sh '''
                        mkdir -p ~/.ssh
                        chmod 700 ~/.ssh
                        ssh-keyscan -H docker >> ~/.ssh/known_hosts

                        ansible-playbook \
                          --inventory hosts.ini \
                          --extra-vars "image=$IMAGE_NAME:$IMAGE_TTL" \
                          playbook.yml
                    '''
                }
            }
        }
    }
}
