pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '21edc252-3633-48c1-a4b7-65513116db6d'
        NETLIFY_AUTH_TOKEN = credentials('netlify')
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t my-playwright .
                '''
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'my-playwright'
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
                            image 'my-playwright'
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'local-report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        // stage('Deploy Staging') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //            npm install netlify-cli node-jq
        //            echo "NETLIFY_SITE_ID: $NETLIFY_SITE_ID"
        //            node_modules/.bin/netlify status
        //            node_modules/.bin/netlify deploy --dir=build
        //            node_modules/.bin/netlify deploy --dir=build --json > staging.json
        //         '''
        //         script {
        //             env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' staging.json", returnStdout: true)
        //         }
        //     }
        // }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'Fake'
            }
            steps {
                sh '''
                echo "NETLIFY_SITE_ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build
                netlify deploy --dir=build --json > staging.json
                CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' staging.json)
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'staging-report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Approval') {
            steps {
                timeout(activity: true, time: 10) {
                    input message: 'Are you ready to deploy', ok: 'Yes'
                }
            }
        }
        // stage('Deploy Prod') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //            npm install netlify-cli
        //            node_modules/.bin/netlify --version
        //            echo "NETLIFY_SITE_ID: $NETLIFY_SITE_ID"
        //            node_modules/.bin/netlify status
        //            node_modules/.bin/netlify deploy --dir=build --prod
        //         '''
        //     }
        // }
        stage('Prod Deploy') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://polite-dieffenbachia-f572a7.netlify.app'
            }
            steps {
                sh '''
                netlify --version
                echo "NETLIFY_SITE_ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --prod
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'prod-report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
