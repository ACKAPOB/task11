pipeline {
    agent any

    stages {
        stage('Run Nginx') {
            steps {
                sh '''
                    # 1. Создаем структуру для volume
                    mkdir -p nginx-data/html
                    echo "<h1>Nginx with Volumes</h1>" > nginx-data/html/index.html

                    # 2. Запускаем контейнер ТОЛЬКО с volume (без проброса портов)
                    docker run -d \
                        --name nginx-vol \
                        -v $(pwd)/nginx-data/html:/usr/share/nginx/html:ro \
                        nginx:alpine
                '''
            }
        }
    }
}
