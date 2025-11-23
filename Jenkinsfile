pipeline {
    agent any

    tools {
        nodejs 'node24'     // must match EXACT name from Jenkins UI
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Hamza525318/react-ci-cd-demo.git'
            }
        }

        stage('Verify Node Version') {
            steps {
                sh 'node -v'
                sh 'npm -v'
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }

    post {
        success { echo 'Build success' }
        failure { echo 'Build failed' }
    }
}
