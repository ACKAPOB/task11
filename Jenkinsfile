pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Copy File') {
            steps {
                sh '''
                    echo "Просто копируем файл index.html"
                    cat index.html
                '''
            }
        }
    }
    
    post {
        always {
            echo "Build status: ${currentBuild.result ?: 'SUCCESS'}"
        }
    }
}
