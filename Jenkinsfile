pipeline {
    agent {
        label 'linux-inbound'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Monitoring project checked out from GitHub'
            }
        }

        stage('Prepare Environment') {
            steps {
                sh '''
                    test -f "$HOME/monitoring-stack/.env"
                    install -m 600 "$HOME/monitoring-stack/.env" .env
                '''
            }
        }

        stage('Validate Ansible') {
            steps {
                sh 'ansible-playbook --syntax-check playbooks/site.yml'
            }
        }

        stage('Validate Docker Compose') {
            steps {
                sh 'docker compose --env-file .env config --quiet'
            }
        }

        stage('Deploy Monitoring') {
            steps {
                sh 'ansible-playbook playbooks/site.yml'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    docker compose \
                      --env-file "$HOME/monitoring-stack/.env" \
                      -f "$HOME/monitoring-stack/docker-compose.yml" ps
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -f .env'
        }
    }
}
