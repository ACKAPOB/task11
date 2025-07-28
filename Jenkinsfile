pipeline {
    agent any
    
    environment {
        TELEGRAM_CHAT_ID = '967851087'
        TELEGRAM_TOKEN = credentials('telegram-creds')
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    try {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/main']],
                            extensions: [],
                            userRemoteConfigs: [[
                                url: 'https://github.com/ACKAPOB/task11.git',
                                credentialsId: '' // добавьте ID credentials если нужно
                            ]]
                        ])
                    } catch (Exception e) {
                        echo "Checkout failed: ${e}"
                        currentBuild.result = 'FAILURE'
                        error('Checkout failed')
                    }
                }
            }
        }
        
        stage('Test Nginx') {
            steps {
                script {
                    // Проверка доступности Docker
                    sh '''
                        if ! command -v docker &> /dev/null; then
                            echo "ERROR: Docker not found!"
                            exit 1
                        fi
                        docker version
                    '''
                    
                    // Запуск тестов
                    try {
                        sh '''
                            docker run --rm -d -p 9889:80 \
                              -v $(pwd)/index.html:/usr/share/nginx/html/index.html:ro \
                              --name nginx-test nginx:stable
                              
                            sleep 5  # Даем Nginx время запуститься
                            
                            # Проверка HTTP 200
                            curl -I http://localhost:9889 | grep '200 OK'
                            
                            # Проверка MD5
                            FILE_MD5=$(md5sum index.html | awk '{print $1}')
                            WEB_MD5=$(curl -s http://localhost:9889 | md5sum | awk '{print $1}')
                            
                            if [ "$FILE_MD5" != "$WEB_MD5" ]; then
                                echo "MD5 mismatch! File: $FILE_MD5, Web: $WEB_MD5"
                                exit 1
                            fi
                            
                            docker stop nginx-test
                        '''
                    } finally {
                        sh 'docker ps -aq | xargs -r docker rm -f || true'
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                def message
                if (currentBuild.result == 'FAILURE') {
                    message = "❌ CI Failed: ${JOB_NAME}\n🔗 ${RUN_DISPLAY_URL}"
                } else {
                    message = "✅ CI Success: ${JOB_NAME}\n🔗 ${RUN_DISPLAY_URL}"
                }
                
                withCredentials([string(credentialsId: 'telegram-creds', variable: 'TOKEN')]) {
                    sh """
                        curl -s -X POST \
                        "https://api.telegram.org/bot${TOKEN}/sendMessage" \
                        -d chat_id=${TELEGRAM_CHAT_ID} \
                        -d text="${message}" \
                        -d parse_mode=HTML
                    """
                }
            }
        }
    }
}
