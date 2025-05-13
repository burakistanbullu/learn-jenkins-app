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
                sh '''
                    ls -la
                    rm -rf node_modules package-lock.json
                    npm init
                    npm install
                    npm run build
                    ls -la
                '''
            }
        }
    }
}
