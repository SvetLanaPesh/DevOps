pipeline {
    agent any
    
    tools {
        ansible 'Ansible'
    }
    
    environment {
        SSH_KEY_PATH = '/tmp/jenkins_ssh_key'
    }
    
    stages {
        stage('🔑 Подготовка SSH ключа') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(
                        credentialsId: 'vagrant-vm-key',
                        keyFileVariable: 'SSH_KEY_FILE'
                    )]) {
                        sh """
                            cp "\$SSH_KEY_FILE" ${SSH_KEY_PATH}
                            chmod 600 ${SSH_KEY_PATH}
                        """
                    }
                }
            }
        }
        
        stage('🐳 Установка Docker') {
            steps {
                dir('ansible-project') {
                    ansiblePlaybook(
                        playbook: 'run_playbook.yml',
                        inventory: 'hosts',
                        tags: 'docker_install',
                        credentialsId: 'vagrant-vm-key',
                        colorized: true
                    )
                }
            }
        }
        
        stage('📂 Клонирование репозиториев') {
            steps {
                dir('ansible-project') {
                    ansiblePlaybook(
                        playbook: 'run_playbook.yml',
                        inventory: 'hosts',
                        tags: 'clone_repos',
                        credentialsId: 'vagrant-vm-key',
                        colorized: true
                    )
                }
            }
        }
        
        stage('🚀 Запуск микросервисов') {
            steps {
                dir('ansible-project') {
                    ansiblePlaybook(
                        playbook: 'run_playbook.yml',
                        inventory: 'hosts',
                        tags: 'services_start',
                        credentialsId: 'vagrant-vm-key',
                        colorized: true
                    )
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Пайплайн выполнен успешно!'
        }
        failure {
            echo '❌ Ошибка выполнения пайплайна'
        }
        always {
            sh "rm -f ${SSH_KEY_PATH} || true"
        }
    }
}
