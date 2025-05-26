pipeline {
    agent any
    
    environment {
        NETLIFY_SITE_ID = '060ae035-cd21-4b63-87ae-ca63465d4ad1'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {

        stage('Docker') {
            steps {
                sh 'docker build -t my-image .'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                    args '-u root'
                }
            }
            environment {
                NPM_CONFIG_CACHE = './.npm-cache'
            }
            steps {
                sh '''
                    echo "Small change to test SCM polling"
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
        stage('Tests') {
            parallel {
                stage('UnitTest') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                            args '-u root'
                        }
                    }

                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                            reuseNode true
                            args '-u root'
                        }
                    }
                    environment {
                        NPM_CONFIG_CACHE = './.npm-cache'
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-image'
                    reuseNode true
                    args '-u root'
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify deploy --dir=build --json > deploy-output.json
                '''
                script {
                    env.STAGING_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        }

         stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                    reuseNode true
                    args '-u root'
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps {
                sh '''
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                    args '-u root'
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
       

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                    reuseNode true
                    args '-u root'
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://tranquil-kleicha-ed9046.netlify.app'
            }

            steps {
                sh '''
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
    }
