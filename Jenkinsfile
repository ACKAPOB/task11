pipeline {
    agent any

    stages {
        stage('Run Nginx') {
            steps {
                sh '''
                    # 1. Создаем директорию и КОПИРУЕМ файл (явно)
                    mkdir -p $WORKSPACE/nginx-content
                    cp $WORKSPACE/index.html $WORKSPACE/nginx-content/
                    
                    # 3. Запускаем контейнер
                    docker run -d \
                        --name nginx_container \
                        -v $WORKSPACE/nginx-content:/usr/share/nginx/html:ro \
                        -p 9889:80 \
                        nginx:alpine
                '''
            }
        }
    }
}
