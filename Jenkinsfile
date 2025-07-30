pipeline {
    agent any

    stages {
        stage('Run Nginx') {
            steps {
                sh '''
                    # 1. Создаем директорию для проекта на хосте
                    mkdir -p $WORKSPACE/nginx-content
                    echo "<h1>Hello from Nginx</h1>" > $WORKSPACE/nginx-content/index.html

                    # 2. Запускаем контейнер с volume (bind mount)
                    docker run -d \
                        --name nginx-vol \
                        -v $WORKSPACE/nginx-content:/usr/share/nginx/html:ro \
                        nginx:alpine
                '''
            }
        }
    }
}
