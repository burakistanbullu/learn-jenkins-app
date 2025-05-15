pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            environment {
                NPM_CONFIG_CACHE = './.npm-cache'
            }
            steps {
                sh '''
                    mkdir -p ./.npm-cache
                    ls -la 
                    node --version
                    npm --version
                    npm install
                    npm run build
                    ls -la
                '''
            }
        }
    }
}

