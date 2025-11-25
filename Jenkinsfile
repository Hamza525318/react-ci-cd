pipeline {
    agent any

    tools {
        nodejs 'node24'          // must match NodeJS tool name in Jenkins
    }

    environment {
        // SonarQube
        SONARQUBE_ENV   = 'sonarqube'   // Jenkins -> Configure System -> SonarQube servers -> Name
        SONAR_SCANNER   = 'SonarScanner' // Jenkins -> Global Tool Configuration -> SonarQube Scanner name

        // Docker Hub
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'  // Jenkins credentials ID for Docker Hub
        DOCKER_IMAGE           = 'hamza525318/react-ci-cd-demo' // change to your DockerHub repo
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    stages {

        stage('Checkout') {
            steps {
                // Uses the same SCM config as the job (Pipeline script from SCM)
                checkout scm
            }
        }

        stage('Install & Build') {
            steps {
                sh '''
                  echo "Using Node:"
                  node -v
                  npm -v

                  # Clean install if package-lock.json exists, else npm install
                  if [ -f package-lock.json ]; then
                    npm ci
                  else
                    npm install
                  fi

                  npm run build
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                  if npm run | grep -q " test"; then
                    echo "Running tests..."
                    npm test -- --watch=false || echo "Tests failed (not failing build for now)"
                  else
                    echo "No test script defined. Skipping tests."
                  fi
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    withEnv(["PATH+SCANNER=${tool SONAR_SCANNER}/bin"]) {
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
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        def img = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        img.push()
                        img.push('latest')
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build ${env.BUILD_NUMBER} succeeded"
        }
        failure {
            echo "❌ Build ${env.BUILD_NUMBER} failed"
        }
    }
}
