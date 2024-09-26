pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'false' // Disable host key checking for simplicity
        ANSIBLE_CONFIG = '.ansible.cfg' // Path to your Ansible config file if needed
        ANSIBLE_PLAYBOOK = 'wordpress.yaml' // Nom du playbook Ansible
        INVENTORY_FILE = 'inventory.ini' // Fichier d'inventaire Ansible
        NODES = 'server' // Groupe d'hôtes ou variables Jenkins
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

        stage('Install Dependencies') {
            steps {
                script {
                    // Installer Ansible et d'autres dépendances
                    sh 'sudo apt-get update'
                    sh 'sudo apt-get install -y ansible python3-pip'
                    sh 'pip3 install docker' // Installer Docker SDK pour Python si nécessaire
                }
            }
        }

        stage('Lint Ansible Playbook') {
            steps {
                script {
                    // Linter le playbook Ansible pour s'assurer qu'il n'y a pas d'erreurs de syntaxe
                    sh 'ansible-lint ${ANSIBLE_PLAYBOOK}'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Exécuter le playbook Ansible pour déployer WordPress
                    sh "ansible-playbook ${ANSIBLE_PLAYBOOK} -i ${INVENTORY_FILE} -e NODES=${NODES}"
                }
            }
        }

        stage('Post-deployment Tests') {
            steps {
                script {
                    // Ajouter des tests pour vérifier le déploiement
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
            // Archiver les logs ou résultats de déploiement
            archiveArtifacts artifacts: '**/playbook.log', allowEmptyArchive: true
        }

        success {
            echo 'Playbook executed successfully.'
        }

        failure {
            echo 'Playbook failed.'
        }
    }
}
