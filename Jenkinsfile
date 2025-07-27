pipeline {
    agent any
    environment {
        // Используем секреты из Jenkins Credentials
        TELEGRAM_TOKEN = credentials('telegram-bot-token')
        TELEGRAM_CHAT_ID = credentials('chat_id')
        NGINX_PORT = "9889"  // Порт для Nginx
    }
    triggers {
        pollSCM('H/5 * * * *')  // Проверка изменений каждые 5 минут
    }
    stages {
        // Этап 1: Запуск Nginx с вашим index.html
        stage('Run Nginx') {
            steps {
                sh '''
                    docker run -d --name nginx-ci \
                    -p ${NGINX_PORT}:80 \
                    -v ${WORKSPACE}/index.html:/usr/share/nginx/html/index.html \
                    cr.yandex/mirror/nginx
                '''
            }
        }

        // Этап 2: Проверка HTTP-ответа
        stage('Check HTTP 200') {
            steps {
                sh '''
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:${NGINX_PORT})
                    [ "$STATUS" -eq 200 ] || { echo "❌ HTTP Status $STATUS (expected 200)"; exit 1; }
                '''
            }
        }

        // Этап 3: Проверка содержимого
        stage('Verify MD5') {
            steps {
                sh '''
                    LOCAL_MD5=$(md5sum index.html | awk "{print \$1}")
                    REMOTE_MD5=$(curl -s http://localhost:${NGINX_PORT} | md5sum | awk "{print \$1}")
                    [ "$LOCAL_MD5" == "$REMOTE_MD5" ] || { echo "❌ MD5 mismatch!"; exit 1; }
                '''
            }
        }
    }
    post {
        // Всегда удаляем контейнер
        always {
            sh 'docker stop nginx-ci || true'
            sh 'docker rm nginx-ci || true'
        }
        // Уведомления в Telegram
        success {
            script {
                sendTelegram("✅ CI Success: ${env.BUILD_URL}")
            }
        }
        failure {
            script {
                sendTelegram("❌ CI Failed: ${env.BUILD_URL}")
            }
        }
    }
}

// Функция для отправки в Telegram
def sendTelegram(message) {
    sh """
        curl -s -X POST "https://api.telegram.org/bot${env.TELEGRAM_TOKEN}/sendMessage" \
        -d "chat_id=${env.CHAT_ID}&text=${message}"
    """
}
