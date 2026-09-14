pipeline {
    agent any

    environment {
        APP_NAME = 'nodejs-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e
                    echo "Installing dependencies..."
                    npm ci
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e
                    echo "Running tests..."
                    npm test
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e
                    echo "Building application..."
                    npm run build
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        set -e
                        echo "Running SonarQube analysis..."
                        sonar-scanner
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e
                    echo "Building Docker image..."

                    docker build \
                        -t ${APP_NAME}:${IMAGE_TAG} \
                        -t ${APP_NAME}:latest \
                        .
                '''
            }
        }

        stage('Docker Test') {
            steps {
                sh '''
                    set -e
                    echo "Checking Docker image..."

                    docker images ${APP_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
            echo "Docker image: ${APP_NAME}:${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed. Please check the Console Output."
        }

        always {
            echo "Cleaning Jenkins workspace..."
            cleanWs()
        }
    }
}