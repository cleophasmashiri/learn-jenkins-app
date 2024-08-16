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
                npm test
                '''
            }
        }
    }
}