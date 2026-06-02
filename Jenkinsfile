pipeline {
    agent { label 'linux-docker' }

    tools {
        nodejs 'node20'
    }

    environment {
        DEBIAN_FRONTEND = 'noninteractive'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            steps {
                sh '''
                    cd src
                    npm ci
                    npm test || true
                '''
            }
        }

        // ВАЖНАЯ СТАДИЯ: Собираем свежий Docker-образ приложения
        stage('Build Image') {
            steps {
                sh 'docker build -t nodejs-demo-app:local -f build/Dockerfile .'
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                sh '''
                    # Сохраняем образ в архив и передаем его через стандартный пайп |
                    #kind load docker-image nodejs-demo-app:local --name desktop
                    
                    # Применяем манифесты в кластер
                    kubectl apply -f mongo-k8s.yaml
                    kubectl apply -f webapp-k8s.yaml
                '''
            }
        }
    }
}
