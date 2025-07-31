pipeline {
    agent any

    stages {
        stage('Run Nginx') {
            steps {
                sh '''
                    # 1. Создаем volume (если его нет)
                    docker volume create nginx_vol

                    # 2. Копируем index.html в volume
                    docker run --rm \
                      -v nginx_vol:/target \
                      -v $WORKSPACE:/source \
                      alpine cp /source/index.html /target/

                    # 3. Запускаем контейнер (без удаления!)
                    docker run -d \
                      --name nginx_container \
                      -v nginx_vol:/usr/share/nginx/html:ro \
                      nginx:alpine
                '''
            }
        }
    }
}
