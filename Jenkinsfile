pipeline {
    agent any

    stages {
        stage('Prepare') {
            steps {
                checkout scm
                sh '''
                    echo "Создаем директории и конфиги для Nginx"
                    mkdir -p nginx/html nginx/conf
                    cp index.html nginx/html/
                    
                    # Минимальный конфиг Nginx (рекомендуемый формат)
                    echo 'server {
                        listen       80;
                        server_name  localhost;

                        location / {
                            root   /usr/share/nginx/html;
                            index  index.html;
                        }
                    }' > nginx/conf/default.conf
                '''
            }
        }

        stage('Run Nginx') {
            steps {
                sh '''
                    echo "Запускаем Nginx с volume"
                    docker run -d \
                        --name nginx-test \
                        -p 9889:80 \
                        -v $WORKSPACE/nginx/html:/usr/share/nginx/html:ro \
                        -v $WORKSPACE/nginx/conf:/etc/nginx/conf.d:ro \
                        nginx:alpine

                    echo "Ждем 3 секунды для инициализации..."
                    sleep 3
                    
                    echo "Проверяем, что контейнер запущен:"
                    docker ps -f name=nginx-test
                    
                    echo "Проверяем доступность:"
                    curl -I http://localhost:9889
                '''
            }
        }
    }

    post {
        always {
            sh '''
                echo "Останавливаем и удаляем контейнер"
                docker stop nginx-test || true
                docker rm nginx-test || true
                
                echo "Очищаем временные файлы"
                rm -rf nginx
            '''
            echo "Build status: ${currentBuild.result ?: 'SUCCESS'}"
        }
    }
}
