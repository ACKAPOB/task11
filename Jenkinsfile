pipeline {
    agent any
    
    environment {
        DOCKER_HOST = "unix:///var/run/docker.sock"
        DOCKER_CLI_EXPERIMENTAL = "enabled"
    }
    
    stages {
        stage('Check Docker') {
            steps {
                script {
                    sh '''
                        # Явная проверка Docker
                        docker --version
                        docker info
                        ls -la /var/run/docker.sock
                    '''
                }
            }
        }
        
        stage('Test Nginx') {
            steps {
                script {
                    try {
                        sh '''
                            # Явное указание пути к docker.sock
                            docker run --rm -d \
                              -v /var/run/docker.sock:/var/run/docker.sock \
                              -p 9889:80 \
                              -v ${WORKSPACE}/index.html:/usr/share/nginx/html/index.html:ro \
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
                            FILE_MD5=$(md5sum index.html | awk '{print $1}')
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
        }
    }
}
