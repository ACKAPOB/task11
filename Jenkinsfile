pipeline {
    agent any
    
    environment {
        DOCKER_HOST = "unix:///var/run/docker.sock"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Prepare Nginx') {
            steps {
                sh '''
                    # Создаем структуру каталогов
                    mkdir -p nginx-setup/conf
                    mkdir -p nginx-setup/html
                    
                    # Копируем наш index.html
                    cp index.html nginx-setup/html/
                    
                    # Создаем конфигурацию Nginx простым echo
                    echo 'worker_processes auto;
                    events {
                        worker_connections 1024;
                    }
                    http {
                        server {
                            listen 9889;
                            root /usr/share/nginx/html;
                            location / {
                                index index.html;
                            }
                        }
                    }' > nginx-setup/conf/nginx.conf
                    
                    # Проверяем созданные файлы
                    echo "### Содержимое конфига ###"
                    cat nginx-setup/conf/nginx.conf
                '''
            }
        }
        
        stage('Run and Test') {
            steps {
                script {
                    try {
                        sh '''
                            # Запуск Nginx
                            docker run -d \
                              --network host \
                              -v $WORKSPACE/nginx-setup/html:/usr/share/nginx/html:ro \
                              -v $WORKSPACE/nginx-setup/conf/nginx.conf:/etc/nginx/nginx.conf:ro \
                              --name nginx-test \
                              nginx:stable
                            
                            # Даем время на запуск
                            sleep 5
                            
                            # Проверка состояния
                            echo "### Состояние контейнера ###"
                            docker ps -a | grep nginx-test
                            
                            # Проверка логов
                            echo "### Логи Nginx ###"
                            docker logs nginx-test
                            
                            # Проверка доступности
                            echo "### Проверка HTTP ###"
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                            echo "HTTP Status Code: $HTTP_CODE"
                            
                            if [ "$HTTP_CODE" != "200" ]; then
                                echo "### Дополнительная диагностика ###"
                                echo "Процессы в контейнере:"
                                docker exec nginx-test ps aux
                                echo "Конфигурация Nginx:"
                                docker exec nginx-test cat /etc/nginx/nginx.conf
                                exit 1
                            fi
                            
                            # Проверка содержимого
                            echo "### Проверка контента ###"
                            curl -s http://localhost:9889 | grep "Версия 1.0" || exit 1
                        '''
                    } finally {
                        sh '''
                            # Очистка
                            echo "### Очистка окружения ###"
                            docker stop nginx-test || true
                            docker rm nginx-test || true
                            rm -rf nginx-setup || true
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "### Итоговый статус: ${currentBuild.result ?: 'SUCCESS'} ###"
        }
    }
}
