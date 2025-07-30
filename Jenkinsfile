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
        
        stage('Prepare Environment') {
            steps {
                sh '''
                    # Создаем рабочую директорию
                    mkdir -p nginx-content
                    cp index.html nginx-content/
                    
                    # Создаем минимальную конфигурацию Nginx
                    echo 'server {
                        listen 80;
                        location / {
                            root /usr/share/nginx/html;
                            index index.html;
                        }
                    }' > nginx.conf
                    
                    # Проверяем файлы
                    ls -la nginx-content/
                    cat nginx.conf
                '''
            }
        }
        
        stage('Run Nginx') {
            steps {
                script {
                    try {
                        sh '''
                            # Запускаем Nginx на стандартном порту 80
                            docker run -d \
                              -p 9889:80 \
                              -v $WORKSPACE/nginx-content:/usr/share/nginx/html:ro \
                              -v $WORKSPACE/nginx.conf:/etc/nginx/conf.d/default.conf:ro \
                              --name nginx-test \
                              nginx:stable
                            
                            # Ждем запуска
                            sleep 5
                            
                            # Проверяем контейнер
                            docker ps -a | grep nginx-test
                            docker logs nginx-test
                            
                            # Проверяем доступность
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                            echo "HTTP Status: $HTTP_CODE"
                            
                            if [ "$HTTP_CODE" != "200" ]; then
                                echo "### Детальная диагностика ###"
                                docker inspect nginx-test
                                docker exec nginx-test nginx -T
                                exit 1
                            fi
                            
                            # Проверка содержимого
                            curl -s http://localhost:9889 | grep "Версия 1.0" || exit 1
                        '''
                    } finally {
                        sh '''
                            # Очистка
                            docker stop nginx-test || true
                            docker rm nginx-test || true
                            rm -f nginx.conf
                            rm -rf nginx-content
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Build status: ${currentBuild.result ?: 'SUCCESS'}"
        }
    }
}
