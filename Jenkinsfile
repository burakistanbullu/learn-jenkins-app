pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                 cleanWs()
                sh '''
                    ls -la
                    npm install
                    npm run build
                    ls -la
                '''
            }
        }
    }
}
