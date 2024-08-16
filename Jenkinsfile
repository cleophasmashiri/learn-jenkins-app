pipeline {
    agent {
        docker {
            image 'node:18-alpine'
        }
    }
    stages {
        stage('Build') {
            sh '''
                ls -la
                node --version
                npm ci
                npm run build
                ls -la
            '''
        }

    }
}