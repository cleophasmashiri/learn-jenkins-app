pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
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
        stage('Testing') {
            steps {
                sh '''
                test -f build/index.html
                npm test
                '''
            }
        }
        stage('e2e tests') {
            agent {
                docker {
                    image 'mcr.mirosoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm i -g serve
                    serve -s build
                    npx playwright test
                '''
            }
        }
    } 
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}