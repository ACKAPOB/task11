pipeline {
    agent any

    stages {
        stage('Run Nginx') {
            steps {
                sh '''
                    # 1. Запускаем Nginx и сразу монтируем файл
                    docker run -d \
                      --name nginx_container \
                      -v $WORKSPACE/index.html:/usr/share/nginx/html/index.html:ro \
                      nginx:alpine
                '''
            }
        }
    }
}
