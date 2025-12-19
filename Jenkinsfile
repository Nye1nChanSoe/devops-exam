pipeline {
    agent any

    tools {
        nodejs "node-24"
    }

    stages {

        stage("Test") {
            steps {
                sh '''
                    node -v
                    npm ci
                    node --test
                '''
            }
        }

        stage("Build") {
            steps {
                sh '''
                    echo "Distribution built..."
                    rm -rf dist
                    mkdir dist
                    cp -r index.js package.json package-lock.json dist/
                '''
            }
        }

        stage("Deploy with Ansible") {
            steps {
                sshagent(credentials: ['secret-key']) {
                    sh '''
                        mkdir -p ~/.ssh
                        chmod 700 ~/.ssh

                        ssh-keyscan -H target >> ~/.ssh/known_hosts
                        chmod 600 ~/.ssh/known_hosts

                        ansible-playbook --inventory hosts.ini playbook.yml
                    '''
                }
            }
        }
    }
}
