pipeline {
    agent any
    
    tools {
        ansible 'Ansible'
    }
    
    environment {
        // Настройки для подключения к ВМ
        VM_IP = '192.168.56.10'
        VM_USER = 'lana25'
        SSH_KEY_PATH = '/tmp/jenkins_ssh_key'
        
        // Настройки репозиториев
        BACKEND_REPO = 'https://github.com/SvetLanaPesh/DevOps-backend.git'
        FRONTEND_REPO = 'https://github.com/SvetLanaPesh/DevOps-frontend.git'
        DEVOPS_REPO = 'https://github.com/SvetLanaPesh/DevOps.git'
    }
    
    stages {
        // Этап 1: Подготовка SSH ключа
        stage('🔑 Подготовка SSH ключа') {
            steps {
                script {
                    echo '=== КОПИРОВАНИЕ SSH КЛЮЧА ДЛЯ ПОДКЛЮЧЕНИЯ К ВМ ==='
                    
                    withCredentials([sshUserPrivateKey(
                        credentialsId: 'vagrant-vm-key',
                        keyFileVariable: 'SSH_KEY_FILE',
                        usernameVariable: 'SSH_USER'
                    )]) {
                        sh """
                            echo 'Копируем SSH ключ из Jenkins credentials...'
                            mkdir -p /tmp
                            cp "\$SSH_KEY_FILE" "${SSH_KEY_PATH}"
                            chmod 600 "${SSH_KEY_PATH}"
                            echo 'Проверяем ключ:'
                            ls -la "${SSH_KEY_PATH}"
                            head -2 "${SSH_KEY_PATH}"
                        """
                    }
                }
            }
        }
        
        // Этап 2: Подготовка окружения на удаленной ВМ
        stage('⚙️ Подготовка окружения') {
            steps {
                script {
                    echo '=== ПОДГОТОВКА ОКРУЖЕНИЯ НА УДАЛЕННОЙ ВМ ==='
                    echo "Подключаемся к ВМ: ${VM_USER}@${VM_IP}"
                    
                    dir('ansible') {
                        // Тестовое подключение к ВМ
                        sh """
                            echo 'Проверяем подключение к ВМ...'
                            ansible -i hosts all -m ping
                        """
                        
                        // Установка Docker и Docker Compose
                        echo 'Устанавливаем Docker и Docker Compose...'
                        ansiblePlaybook(
                            playbook: 'run_playbook.yml',
                            inventory: 'hosts',
                            tags: 'docker_install',
                            credentialsId: 'vagrant-vm-key',
                            colorized: true,
                            extraVars: [
                                'vm_ip': env.VM_IP,
                                'vm_user': env.VM_USER
                            ]
                        )
                        
                        // Клонирование репозиториев
                        echo 'Клонируем репозитории...'
                        ansiblePlaybook(
                            playbook: 'run_playbook.yml',
                            inventory: 'hosts',
                            tags: 'clone_repos',
                            credentialsId: 'vagrant-vm-key',
                            colorized: true,
                            extraVars: [
                                'backend_repo': env.BACKEND_REPO,
                                'frontend_repo': env.FRONTEND_REPO,
                                'devops_repo': env.DEVOPS_REPO
                            ]
                        )
                    }
                }
            }
        }
        
        // Этап 3: Запуск микросервисов
        stage('🚀 Запуск микросервисов') {
            steps {
                script {
                    echo '=== ЗАПУСК BACKEND И FRONTEND ЧЕРЕЗ DOCKER COMPOSE ==='
                    
                    dir('ansible') {
                        ansiblePlaybook(
                            playbook: 'run_playbook.yml',
                            inventory: 'hosts',
                            tags: 'start_services',
                            credentialsId: 'vagrant-vm-key',
colorized: true,
                            extraVars: [
                                'compose_file_path': '/home/lana25/microservices/docker-compose.yml'
                            ]
                        )
                    }
                }
            }
        }
        
        // Этап 4: Проверка работоспособности
        stage('✅ Проверка работоспособности') {
            steps {
                script {
                    echo '=== ПРОВЕРКА ДОСТУПНОСТИ СЕРВИСОВ ==='
                    
                    dir('ansible') {
                        // Проверка бекенда
                        sh """
                            echo 'Проверяем доступность бекенда...'
                            ansible -i hosts all -m shell \\
                                -a "curl -s -o /dev/null -w '%{http_code}' http://localhost:5000/data || echo '500'"
                        """
                        
                        // Проверка фронтенда
                        sh """
                            echo 'Проверяем доступность фронтенда...'
                            ansible -i hosts all -m shell \\
                                -a "curl -s -o /dev/null -w '%{http_code}' http://localhost:80 || echo '500'"
                        """
                        
                        // Проверка работающих контейнеров
                        sh """
                            echo 'Проверяем статус контейнеров...'
                            ansible -i hosts all -m shell \\
                                -a "cd /home/lana25/microservices && docker-compose ps"
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'ПАЙПЛАЙН УСПЕШНО ВЫПОЛНЕН!'
            echo ''
            echo 'РЕЗУЛЬТАТЫ РАЗВЕРТЫВАНИЯ:'
            echo '============================='
            echo 'Docker и Docker Compose установлены'
            echo 'Репозитории успешно клонированы'
            echo 'Микросервисы запущены через Docker Compose'
            echo ''
            echo 'СЕРВИСЫ ДОСТУПНЫ ПО АДРЕСАМ:'
            echo '==============================='
            echo 'Frontend: http://${VM_IP}:80'
            echo 'Backend API: http://${VM_IP}:5000/data'
            echo ''
            echo 'Для повторного развертывания просто запустите эту джобу снова'
        }
        
        failure {
            echo 'ПАЙПЛАЙН ЗАВЕРШИЛСЯ С ОШИБКОЙ'
            echo ''
            echo 'ВОЗМОЖНЫЕ ПРИЧИНЫ:'
            echo '====================='
            echo '1. ВМ не запущена или недоступна'
            echo '2. Неправильные SSH credentials'
            echo '3. Проблемы с сетью'
            echo '4. Ошибки в Ansible playbook'
            echo ''
            echo 'Проверьте Console Output для деталей'
        }
        
        always {
            echo 'Очистка временных файлов...'
            sh """
                rm -f "${SSH_KEY_PATH}" || true
                echo 'Временные файлы удалены'
            """
            
            // Сохранение логов Ansible
            archiveArtifacts artifacts: 'ansible/**/*.log', allowEmptyArchive: true
        }
    }
}
