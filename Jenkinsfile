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
                    mkdir -p nginx-content
                    cp index.html nginx-content/
                    
                    # Создаем полную конфигурацию Nginx
                    cat > nginx-content/nginx.conf << 'EOF'
                    events {
                        worker_connections 1024;
                    }
                    http {
                        server {
                            listen 9889;
                            location / {
                                root /usr/share/nginx/html;
                                index index.html;
                            }
                        }
                    }
                    EOF
                    
                    chmod -R a+r nginx-content
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
                              -v $WORKSPACE/nginx-content:/usr/share/nginx/html:ro \
                              -v $WORKSPACE/nginx-content/nginx.conf:/etc/nginx/nginx.conf:ro \
                              --name nginx-test \
                              nginx:stable
                            
                            echo "### Ждем запуска ###"
                            sleep 5
                            
                            echo "### Проверка конфигурации ###"
                            docker exec nginx-test nginx -T
                            
                            echo "### Проверка портов ###"
                            docker exec nginx-test netstat -tulnp || docker exec nginx-test ss -tulnp
                            
                            echo "### Проверка доступности ###"
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                            echo "HTTP Code: $HTTP_CODE"
                            
                            if [ "$HTTP_CODE" != "200" ]; then
                                echo "### Логи Nginx ###"
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
                            rm -rf nginx-content || true
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
