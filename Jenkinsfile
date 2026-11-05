pipeline {
    agent {
        label 'linux-inbound'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning monitoring project from GitHub'

                withCredentials([usernamePassword(
                    credentialsId: 'github',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh '''
                        set +x
                        git -c credential.helper='!f() { echo username="$GIT_USERNAME"; echo password="$GIT_TOKEN"; }; f' \
                        clone https://github.com/Aditya123-kumar/ansible-monitoring.git .
                    '''
                }
            }
        }

        stage('Prepare Environment') {
            steps {
                sh '''
                    set -eu

                    if [ ! -f "$HOME/monitoring-stack/.env" ]; then
                        echo "ERROR: Monitoring .env file not found"
                        exit 1
                    fi

                    install -m 600 "$HOME/monitoring-stack/.env" .env

                    echo "Environment file prepared securely"
                '''
            }
        }

        stage('Validate Ansible') {
            steps {
                sh '''
                    ansible-playbook --syntax-check playbooks/site.yml
                '''
            }
        }

        stage('Validate Docker Compose') {
            steps {
                sh '''
                    docker compose --env-file .env config --quiet
                '''
            }
        }

        stage('Deploy Monitoring') {
            steps {
                sh '''
                    ansible-playbook playbooks/site.yml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    docker compose \
                      --env-file "$HOME/monitoring-stack/.env" \
                      -f "$HOME/monitoring-stack/docker-compose.yml" \
                      ps
                '''
            }
        }
    }

    post {
        always {
            script {
                if (env.WORKSPACE) {
                    sh '''
                        rm -f .env || true
                    '''
                }
            }
        }

        success {
            echo 'Monitoring deployment completed successfully!'
        }

        failure {
            echo 'Monitoring deployment failed. Please check the console output.'
        }
    }
}

