pipeline {
    agent { label 'master' } // Или ваш агент, если он не master

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                // Этот шаг уже есть, он не всегда явно нужен в Declarative Pipeline,
                // так как Jenkinsfile сам по себе извлекается из SCM.
                // Но если вы хотите гарантировать, что рабочая директория будет очищена,
                // можно добавить: cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                // Если вы используете один и тот же репозиторий, этот второй checkout
                // обычно избыточен, так как содержимое уже есть в workspace
                // после 'Declarative: Checkout SCM'.
                // Однако, если это было сделано намеренно для какой-то специфики,
                // оставьте его.
                checkout scm
            }
        }

        stage('Prepare Workspace') {
            steps {
                sh '''
                    echo "### Подготовка workspace ###"
                    mkdir -p nginx-content
                    cp index.html nginx-content/
                    chmod -R a+rx nginx-content
                    echo "### Содержимое index.html ###"
                    cat nginx-content/index.html
                '''
            }
        }

        stage('Test Nginx') {
            steps {
                script {
                    echo "### Запуск Nginx в сетевом режиме host ###"
                    // Добавим --rm, чтобы контейнер автоматически удалялся после остановки
                    // (но мы все равно будем останавливать его явно в post-build)
                    sh "docker run -d --network host -v ${workspace}/nginx-content:/usr/share/nginx/html:ro --name nginx-test nginx:stable"

                    echo "### Проверка состояния контейнера (сразу после запуска) ###"
                    sh "docker ps -a | grep nginx-test"

                    echo "### Ждем немного, пока Nginx полностью стартует... ###"
                    sleep 10 // Увеличил время ожидания, чтобы Nginx успел запуститься
