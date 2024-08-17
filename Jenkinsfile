pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '21edc252-3633-48c1-a4b7-65513116db6d'
        NETLIFY_AUTH_TOKEN = credentials('netlify')
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Running Testing') {
            parallel {
                stage('Testing') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        test -f build/index.html
                        npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('e2e tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-dev-report', reportFiles: 'index.html', reportName: 'e2e-report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
         stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli
                   node_modules/.bin/netlify --version
                   echo "NETLIFY_SITE_ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }
        stage(''Approval) {
            steps {
                input message: 'Are you ready to deploy', ok: 'Yes'
            }
        }
        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli
                   node_modules/.bin/netlify --version
                   echo "NETLIFY_SITE_ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod e2e tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment {
                        CI_ENVIRONMENT_URL = 'https://polite-dieffenbachia-f572a7.netlify.app'
                    }
                    steps {
                        sh '''
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-prod-report', reportFiles: 'index.html', reportName: 'e2e-report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
        }
    }
}
