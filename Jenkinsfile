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
        
        stage('Prepare Workspace') {
            steps {
                sh '''
                    echo "### Подготовка workspace ###"
                    mkdir -p nginx-config
                    mkdir -p nginx-content
                    
                    # Копируем наш index.html
                    cp index.html nginx-content/
                    
                    # Создаем кастомную конфигурацию Nginx
                    echo 'server {
                        listen 9889;
                        server_name localhost;
                        location / {
                            root /usr/share/nginx/html;
                            index index.html;
                        }
                    }' > nginx-config/nginx-custom.conf
                    
                    chmod -R a+rx nginx-content nginx-config
                '''
            }
        }
        
        stage('Test Nginx') {
            steps {
                script {
                    try {
                        sh '''
                            echo "### Запуск Nginx с кастомной конфигурацией ###"
                            docker run -d \
                              --network host \
                              -v $WORKSPACE/nginx-content:/usr/share/nginx/html:ro \
                              -v $WORKSPACE/nginx-config:/etc/nginx/conf.d:ro \
                              --name nginx-test \
                              nginx:stable
                            
                            echo "### Проверка состояния контейнера ###"
                            docker ps -a | grep nginx-test
                            sleep 5
                            docker logs nginx-test
                            
                            echo "### Проверка конфигурации Nginx ###"
                            docker exec nginx-test nginx -T | grep 9889
                            
                            echo "### Проверка доступности ###"
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                            echo "HTTP Code: $HTTP_CODE"
                            
                            if [ "$HTTP_CODE" != "200" ]; then
                                echo "### Детальная диагностика ###"
                                docker exec nginx-test netstat -tulnp || docker exec nginx-test ss -tulnp
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
                            rm -rf nginx-content nginx-config || true
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
