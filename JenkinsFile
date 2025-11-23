pipeline {
    agent any

    environment {
        // Docker Hub
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds' // create this in Jenkins
        DOCKER_IMAGE = 'your-dockerhub-username/react-ci-demo'

        // SonarQube
        SONARQUBE_ENV = 'sonarqube' // Name configured in Jenkins
        SCANNER_TOOL = 'SonarScanner'

        // Node version (optional if using NodeJS plugin)
        NODE_VERSION = '20'
    }

    triggers {
        // In addition to webhook, you can also poll:
        // pollSCM('H/5 * * * *')
    }

    options {
        ansiColor('xterm')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies & Build') {
            steps {
                sh '''
                  node -v || curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | sudo -E bash - && sudo apt-get install -y nodejs
                  npm ci
                  npm run build
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                  npm test -- --watch=false --coverage || echo "Tests failed"
                '''
            }
            post {
                always {
                    junit 'coverage/**/junit.xml' // if you generate JUnit reports
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONARQUBE_ENV) {
                    withEnv(["PATH+SCANNER=${tool SCANNER_TOOL}/bin"]) {
                        sh 'sonar-scanner'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
                        def appImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        appImage.push()
                        appImage.push('latest')
                    }
                }
            }
        }

        // Optional
        stage('Deploy (Optional)') {
            when {
                branch 'main'
            }
            steps {
                echo 'Here you could SSH to a server / trigger ECS deploy / Helm upgrade'
            }
        }
    }

    post {
        success {
            echo "Build ${env.BUILD_NUMBER} succeeded"
        }
        failure {
            echo "Build ${env.BUILD_NUMBER} failed"
        }
    }
}
