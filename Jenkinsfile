pipeline {
    agent any
    
    triggers {
        githubPush()
    }
    
    environment {
        TELEGRAM_CHAT_ID = '967851087'
        TELEGRAM_TOKEN = credentials('telegram-creds') // Настроить в Jenkins
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test Nginx') {
            steps {
                script {
                    docker.image('nginx:stable').withRun(
                        "-p 9889:80 -v ${WORKSPACE}/index.html:/usr/share/nginx/html/index.html:ro"
                    ) { container ->
                        // Проверка HTTP 200
                        sh "curl -I http://localhost:9889 | grep '200 OK'"
                        
                        // Проверка MD5
                        def fileMd5 = sh(
                            script: "md5sum index.html | awk '{print \$1}'", 
                            returnStdout: true
                        ).trim()
                        
                        def webMd5 = sh(
                            script: "curl -s http://localhost:9889 | md5sum | awk '{print \$1}'", 
                            returnStdout: true
                        ).trim()
                        
                        if (fileMd5 != webMd5) {
                            error "MD5 mismatch! File: ${fileMd5}, Web: ${webMd5}"
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Очистка контейнеров
            sh 'docker ps -aq | xargs -r docker rm -f || true'
        }
        failure {
            // Telegram уведомление
            sh """
                curl -s -X POST \
                "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
                -d chat_id=${TELEGRAM_CHAT_ID} \
                -d text="❌ CI Failed: ${JOB_NAME}%0A🔗 ${RUN_DISPLAY_URL}" \
                -d parse_mode=HTML
            """
        }
        success {
            sh """
                curl -s -X POST \
                "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
                -d chat_id=${TELEGRAM_CHAT_ID} \
                -d text="✅ CI Success: ${JOB_NAME}%0A🔗 ${RUN_DISPLAY_URL}" \
                -d parse_mode=HTML
            """
        }
    }
}
