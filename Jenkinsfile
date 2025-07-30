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
                    echo "### Подготовка Nginx ###"
                    # Создаем структуру каталогов
                    mkdir -p nginx-setup/conf
                    mkdir -p nginx-setup/html
                    
                    # Копируем наш index.html
                    cp index.html nginx-setup/html/
                    
                    # Создаем полную конфигурацию Nginx
                    cat > nginx-setup/conf/nginx.conf << 'EOF'
                    worker_processes auto;
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
                    }
                    EOF
                    
                    # Проверяем созданные файлы
                    ls -la nginx-setup/
                    ls -la nginx-setup/conf/
                    ls -la nginx-setup/html/
                '''
            }
        }
        
        stage('Run and Test') {
            steps {
                script {
                    try {
                        sh '''
                            echo "### Запуск Nginx ###"
                            docker run -d \
                              --network host \
                              -v $WORKSPACE/nginx-setup/html:/usr/share/nginx/html:ro \
                              -v $WORKSPACE/nginx-setup/conf:/etc/nginx:ro \
                              --name nginx-test \
                              nginx:stable
                            
                            echo "### Ждем запуска ###"
                            sleep 5
                            
                            echo "### Проверка конфигурации ###"
                            docker exec nginx-test nginx -T
                            
                            echo "### Проверка процессов ###"
                            docker exec nginx-test ps aux | grep nginx
                            
                            echo "### Проверка доступности ###"
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                            echo "HTTP Code: $HTTP_CODE"
                            
                            if [ "$HTTP_CODE" != "200" ]; then
                                echo "### Детальная диагностика ###"
                                docker exec nginx-test netstat -tulnp || docker exec nginx-test ss -tulnp
                                docker logs nginx-test
                                exit 1
                            fi
                            
                            echo "### Проверка содержимого ###"
                            curl -s http://localhost:9889 | grep "Версия 1.0" || exit 1
                        '''
                    } finally {
                        sh '''
                            echo "### Очистка ###"
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
            echo "### Build status: ${currentBuild.result ?: 'SUCCESS'} ###"
        }
    }
}
