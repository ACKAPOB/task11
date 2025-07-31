pipeline {
    agent any

    stages {
        stage('Checkout & Prepare') {
            steps {
                checkout scm  // Клонирует репозиторий
            }
        }

        stage('Run Nginx') {
            steps {
                sh '''
                    # Запускаем Nginx с пробросом портов
                    docker run -d \
                        --name nginx_ci \
                        -v $WORKSPACE/index.html:/usr/share/nginx/html/index.html:ro \
                        -p 9889:80 \
                        nginx:alpine
                    
                    sleep 3  // Ждем запуска
                '''
            }
        }

        stage('Test HTTP 200') {
            steps {
                sh '''
                    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                    if [ "$HTTP_CODE" != "200" ]; then
                        echo "❌ Ошибка: Nginx вернул код $HTTP_CODE вместо 200"
                        exit 1
                    else
                        echo "✅ HTTP-200: Страница доступна"
                    fi
                '''
            }
        }

        stage('Test MD5') {
            steps {
                sh '''
                    LOCAL_MD5=$(md5sum index.html | awk '{print $1}')
                    REMOTE_MD5=$(curl -s http://localhost:9889 | md5sum | awk '{print $1}')

                    if [ "$LOCAL_MD5" != "$REMOTE_MD5" ]; then
                        echo "❌ Ошибка: MD5 не совпадает (локальный: $LOCAL_MD5, удаленный: $REMOTE_MD5)"
                        exit 1
                    else
                        echo "✅ MD5 совпадает: $LOCAL_MD5"
                    fi
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f nginx_ci || true'  // Удаляем контейнер
        }
        failure {
            // Здесь можно добавить уведомление в Slack/Telegram
            echo "🚨 CI Failed! Проверьте изменения в index.html"
        }
    }
}
