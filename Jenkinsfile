pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18'
                }
            }
            steps {
                sh '''
                    rm -rf node_modules package-lock.json
                    npm cache clean --force
                    npm install
                    npm run build
                '''
            }
        }
    }
}
