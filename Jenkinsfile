pipeline {
    agent any
    
    tools {
        ansible 'Ansible'
    }
    
    stages {
        // Этап 1: Подготовка окружения
        stage('Подготовка окружения') {
            steps {
                script {
                    echo '=== КЛОНИРОВАНИЕ РЕПОЗИТОРИЕВ ==='
                    
                    dir('ansible-project') {
                        // Клонируем ваш DevOps репозиторий
                        git branch: 'main',
                            url: 'https://github.com/SvetLanaPesh/DevOps.git'
                        
                        // Запускаем Ansible для установки Docker и клонирования репозиториев
                        ansiblePlaybook(
                            playbook: 'run_playbook.yml',
                            inventory: 'hosts',
                            tags: 'install,deploy',
                            credentialsId: 'vagrant-vm-key',
                            colorized: true
                        )
                    }
                }
            }
        }
        
        // Этап 2: Запуск приложений
        stage('Запуск микросервисов') {
            steps {
                script {
                    echo '=== ЗАПУСК BACKEND И FRONTEND ==='
                    
                    dir('ansible-project') {
                        // Запускаем только задачи старта сервисов
                        ansiblePlaybook(
                            playbook: 'run_playbook.yml',
                            inventory: 'hosts',
                            tags: 'start',
                            credentialsId: 'vagrant-vm-key',
                            colorized: true
                        )
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'ПАЙПЛАЙН УСПЕШНО ВЫПОЛНЕН!'
            echo 'Frontend доступен по: http://localhost:8080'
            echo 'Backend API: http://localhost:5001/data'
        }
        failure {
            echo 'ПАЙПЛАЙН ЗАВЕРШИЛСЯ С ОШИБКОЙ'
            // Можно добавить уведомления
        }
    }
}
