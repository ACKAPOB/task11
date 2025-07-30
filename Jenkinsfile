pipeline {
    agent any
    
    environment {
        DOCKER_HOST = "unix:///var/run/docker.sock"
    }
    
    stages {
        stage('Check Docker') {
            steps {
                script {
                    sh '''
                        docker --version
                        docker info
                        ls -la /var/run/docker.sock
                    '''
                }
            }
        }
        
        stage('Prepare Workspace') {
            steps {
                sh '''
                    # Создаем директорию для nginx
                    mkdir -p nginx-content
                    cp index.html nginx-content/
                    chmod -R a+rwx nginx-content
                '''
            }
        }
        
        stage('Test Nginx') {
            steps {
                script {
                    try {
                        sh '''
                            # Запускаем Nginx с правильными путями
                            docker run --rm -d \
                              -p 9889:80 \
                              -v ${WORKSPACE}/nginx-content:/usr/share/nginx/html:ro \
                              --name nginx-test \
                              nginx:stable
                            
                            sleep 5
                            
                            # Проверка HTTP 200
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9889)
                            if [ "$HTTP_CODE" != "200" ]; then
                                echo "ERROR: HTTP response code $HTTP_CODE (expected 200)"
                                exit 1
                            fi
                            
                            # Проверка MD5
                            FILE_MD5=$(md5sum nginx-content/index.html | awk '{print $1}')
                            WEB_MD5=$(curl -s http://localhost:9889 | md5sum | awk '{print $1}')
                            
                            if [ "$FILE_MD5" != "$WEB_MD5" ]; then
                                echo "ERROR: MD5 mismatch! File: $FILE_MD5, Web: $WEB_MD5"
                                exit 1
                            fi
                        '''
                    } finally {
                        sh 'docker stop nginx-test || true'
                        sh 'docker rm nginx-test || true'
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Build status: ${currentBuild.result ?: 'SUCCESS'}"
            sh 'rm -rf nginx-content || true'
        }
    }
}
