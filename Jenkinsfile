pipeline {
    agent any

    environment {
          // Nom du playbook Ansible
                            // Fichier d'inventaire Ansible
                             // Groupe d'hôtes ou variables Jenkins
        ANSIBLE_HOST_KEY_CHECKING = 'false' // Disable host key checking for simplicity
        ANSIBLE_CONFIG = '.ansible.cfg' // Path to your Ansible config file if needed
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Vérifier le dépôt contenant le playbook Ansible
                    checkout scm
                }
            }
        }

        stage('Install Ansible') {
            steps {
                script {
                    // Installer Ansible s'il n'est pas déjà installé
                    sh 'apt update && sudo apt install -y ansible'
                }
            }
        }
        
        stage('deploye') {
            steps {
                script {
                    // Installer Ansible s'il n'est pas déjà installé
                    sh 'ansible-playbook wordpress.yaml --ask-pass -e NODES=server'
                }
            }
        }

        stage('Run Playbook') {
            steps {
                script {
                    // Exécuter le playbook Ansible pour déployer la stack WordPress et MariaDB
                    sh """
                    ansible-playbook ${ANSIBLE_PLAYBOOK} \
                    -i ${hosts} \
                    -e NODES=${NODES} \
                    """
            steps {
                // Checkout the code from your repository
                git url: 'https://github.com/ELkab007/CI-CD-Jenkins.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Ansible and any other dependencies
                sh 'sudo apt-get update'
                sh 'sudo apt-get install -y ansible python3-pip'
                sh 'pip3 install docker' // Install Docker SDK for Python if needed
            }
        }

        stage('Lint Ansible Playbook') {
            steps {
                // Lint the Ansible playbook to ensure there are no syntax errors
                sh 'ansible-lint wordpress.yaml'
            }
        }

        stage('Deploy') {
            steps {
                // Run the Ansible playbook to deploy WordPress
                sh 'ansible-playbook wordpress.yaml -e "NODES=server"'
            }
        }

        stage('Post-deployment Tests') {
            steps {
                // Here you can add tests to verify the deployment
                script {
                    def response = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://192.168.122.128:8082', returnStdout: true).trim()
                    if (response != '200') {
                        error "Deployment failed: HTTP response code ${response}"
                    } else {
                        echo "Deployment successful: HTTP response code ${response}"
                    }
                }
            }
        }
    }

    post {
        always {
            // Archive les logs ou résultats de déploiement
            archiveArtifacts artifacts: '**/playbook.log', allowEmptyArchive: true
        }

        success {
            echo 'Playbook executed successfully.'
        }

        failure {
            echo 'Playbook failed.'
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
