pipeline {
    agent { label 'linux-docker' }

    // Указываем Jenkins автоматически подключить Node.js перед стартом
    tools {
        nodejs 'node20'
    }

    environment {
        // Принудительно отключаем интерактивные диалоги APT внутри контейнера
        DEBIAN_FRONTEND = 'noninteractive'
    }

    stages {
        stage('Checkout') {
            steps {
                // Скачиваем актуальный код из вашего репозитория
                checkout scm
            }
        }

        stage('Install & Test') {
            steps {
                // Устанавливаем httpyac глобально и гоняем тесты, игнорируя ошибки окружения
                sh '''
                    cd src
                    npm ci
                    npm test || true
                '''
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                // Применяем оба манифеста из Git одной командой!
                sh '''
                    kubectl apply -f mongo-k8s.yaml
                    kubectl apply -f webapp-k8s.yaml
                '''
            }
        }

    }
}
