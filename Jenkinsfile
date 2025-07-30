pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Run Nginx') {
            steps {
                sh '''
                    echo "Запускаем Nginx в контейнере"
                    docker run --name nginx-test -d -p 9889:80 nginx:alpine
                    
                    echo "Ждем 5 секунд для инициализации"
                    sleep 5
                    
                    echo "Проверяем, что контейнер запустился"
                    docker ps | grep nginx-test
                    
                    echo "Проверяем доступность Nginx"
                    curl -v http://localhost:9889
                '''
            }
        }
        
        stage('Cleanup') {
            steps {
                sh '''
                    echo "Останавливаем и удаляем контейнер"
                    docker stop nginx-test || true
                    docker rm nginx-test || true
                '''
            }
        }
    }
    
    post {
        always {
            echo "Build status: ${currentBuild.result ?: 'SUCCESS'}"
        }
    }
}
