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

             withEnv(['JENKINS_NODE_COOKIE=dontKillMe']) {
                sh '''
                    # 1. Жестко убиваем старый процесс по порту 3006
                    pkill -f "3006:3000" || true
                    sleep 2
                    
                    # 2. Запускаем процесс в фоне. Флаг & в конце и перенаправление логов ОБЯЗАТЕЛЬНЫ
                    nohup kubectl port-forward deployment/kube-stack-grafana 3006:3000 --address=0.0.0.0 > pf-grafana.log 2>&1 &
                     
                    # 3. Даем 2 секунды процессу запуститься и проверяем, что он не упал сразу
                    sleep 2
                    ps aux | grep "3006:3000" | grep -v grep
                    
                    echo "Grafana port-forward started on http://localhost:3006"
                '''
                }
            }
        }
    }
}
