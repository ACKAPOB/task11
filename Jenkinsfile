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
        
        stage('Verify Files') {
            steps {
                sh '''
                    echo "### Проверка файлов ###"
                    ls -la $WORKSPACE/
                    ls -la $WORKSPACE/index.html
                    chmod a+r $WORKSPACE/index.html
                '''
            }
        }
        
        stage('Test Nginx') {
            steps {
                script {
                    try {
                        sh '''
                            echo "### Запуск Nginx ###"
                            docker run -d \
                              -p 9889:80 \
                              -v $WORKSPACE/index.html:/usr/share/nginx/html/index.html:ro \
                              --name nginx-test \
                              nginx:stable
                            
                            sleep 5
                        '''
                        
                        sh '''
                            echo "### Проверка доступности ###"
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                            echo "HTTP Code: $HTTP_CODE"
                            
                            if [ "$HTTP_CODE" != "200" ]; then
                                echo "### Дополнительная диагностика ###"
                                docker ps -a
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
