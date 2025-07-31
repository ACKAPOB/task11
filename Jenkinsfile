pipeline {
    agent any

    stages {
        stage('Run Nginx') {
            steps {
                sh '''
                    # 1. Создаем директорию для Nginx
                    mkdir -p $WORKSPACE/nginx-content
                    cp index.html $WORKSPACE/nginx-content/

                    # 2. Запускаем Nginx (без удаления старого)
                    docker run -d \
                      --name nginx_container \
                      -v $WORKSPACE/nginx-content:/usr/share/nginx/html:ro \
                      -p 9889:80 \
                      nginx:alpine

                    echo "Если нужно перезапустить, сначала выполните:"
                    echo "docker rm -f nginx_container"
                '''
            }
        }
    }
}
