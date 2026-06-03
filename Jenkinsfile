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
                    docker save nodejs-demo-app:local | docker exec -i desktop-control-plane ctr -n=k8s.io images import -
                    
                    # Применяем манифесты в кластер
                    kubectl apply -f mongo-k8s.yaml
                    kubectl apply -f webapp-k8s.yaml
                    kubectl apply -f app-servicemonitor.yaml
                '''
                sh '''
                    # 1. Убиваем старый процесс проброса Grafana, если он остался от прошлой сборки
                    pkill -f "kubectl port-forward deployment/kube-stack-grafana" || true
                    sleep 2
                    
                    # 2. Прорубаем туннель из изолированной сети k8s в подсеть WSL (172.25)
                    # Перенаправляем логи в файл, чтобы команда nohup отпустила консоль Jenkins
                    nohup kubectl port-forward deployment/kube-stack-grafana 3006:3000 --address=0.0.0.0 > pf-grafana.log 2>&1 &
                    
                    echo "Grafana port-forward started on http://localhost:3006"
                '''
            }
        }
    }
}
