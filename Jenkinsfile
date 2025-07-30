pipeline {
    agent any
    
    stages {
        stage('Run Nginx') {
            steps {
                sh '''
                    echo "Запускаем Nginx в контейнере"
                    docker run --name nginx-test -d -p 9889:80 nginx:alpine
                    
                    echo "Даем время на запуск"
                    sleep 5
                    
                    echo "Проверяем контейнер"
                    docker ps
                '''
            }
        }
    }
    
    post {
        always {
            sh '''
                echo "Убираем контейнер"
                docker stop nginx-test || true
                docker rm nginx-test || true
            '''
            echo "Build status: ${currentBuild.result ?: 'SUCCESS'}"
        }
    }
}
