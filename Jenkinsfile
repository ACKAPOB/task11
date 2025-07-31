pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare & Run Nginx') {
            steps {
                sh '''
                    # 1. Создаем директорию и копируем index.html
                    mkdir -p $WORKSPACE/nginx-html
                    cp $WORKSPACE/index.html $WORKSPACE/nginx-html/

                    # 2. Запускаем Nginx с монтированием ДИРЕКТОРИИ (не файла!)
                    docker run -d \
                        --name nginx_ci \
                        -v $WORKSPACE/nginx-html:/usr/share/nginx/html:ro \
                        -p 9889:80 \
                        nginx:alpine

                    sleep 3  # Ждем запуска
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
                        echo "✅ HTTP-200: OK"
                    fi
                '''
            }
        }

        stage('Test MD5') {
            steps {
                sh '''
                    LOCAL_MD5=$(md5sum $WORKSPACE/nginx-html/index.html | awk '{print $1}')
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
            sh 'docker rm -f nginx_ci || true'
            sh 'rm -rf $WORKSPACE/nginx-html'
        }
        failure {
            echo "🚨 CI Failed! Подробности в логах."
            // Добавьте здесь уведомление в Telegram/Slack (как в прошлом примере)
        }
    }
}
