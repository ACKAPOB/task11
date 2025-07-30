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
                    mkdir -p nginx-content
                    cp index.html nginx-content/
                    chmod -R a+rx nginx-content
                    echo "### Содержимое index.html ###"
                    cat nginx-content/index.html
                '''
            }
        }
        
        stage('Test Nginx') {
            steps {
                script {
                    try {
                        sh '''
                            echo "### Запуск Nginx в сетевом режиме host ###"
                            docker run -d \
                              --network host \
                              -v $WORKSPACE/nginx-content:/usr/share/nginx/html:ro \
                              --name nginx-test \
                              nginx:stable
                            
                            echo "### Проверка состояния контейнера ###"
                            docker ps -a | grep nginx-test
                            sleep 5
                            docker logs nginx-test
                            
                            echo "### Проверка доступности ###"
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                            echo "HTTP Code from localhost: $HTTP_CODE"
                            
                            EXTERNAL_IP=$(curl -s ifconfig.me)
                            echo "### Проверка внешнего доступа к $EXTERNAL_IP:9889 ###"
                            EXT_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://$EXTERNAL_IP:9889)
                            echo "HTTP Code from external: $EXT_CODE"
                            
                            if [ "$HTTP_CODE" != "200" ] || [ "$EXT_CODE" != "200" ]; then
                                echo "### Детальная диагностика ###"
                                netstat -tulnp || ss -tulnp
                                docker inspect nginx-test
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
