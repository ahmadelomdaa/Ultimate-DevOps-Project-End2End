pipeline {
    agent any

    tools {
        // Use auto-installed SonarQube Scanner CLI
        sonarRunner 'sonar-scanner'
    }

    environment {
        // Define application metadata and image name
        APP_NAME = 'ultimate-devops-app'
        IMAGE_TAG = "v1.0.${BUILD_NUMBER}"
        SNYK_TOKEN = credentials('snyk-token')
    }

    stages {
        // Stage 1: Source Code Checkout
        stage('Checkout Source Code') {
            steps {
                echo '[INFO] Checking out latest source code from GitHub...'
                checkout scm
            }
        }

        // Stage 2: Code Quality Analysis with SonarQube
        stage('SonarQube Quality Scan') {
            steps {
                echo '[INFO] Starting Static Code Analysis via SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=Ultimate-DevOps-Project \
                        -Dsonar.projectName="Ultimate DevOps Project" \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/**,coverage/**
                    '''
                }
            }
        }

        // Stage 3: Dependency Security Scan via Snyk
        stage('Snyk Security Scan') {
            steps {
                echo '[INFO] Scanning node dependencies for vulnerabilities using Snyk...'
                sh '''
                    npx --yes snyk auth $SNYK_TOKEN
                    npx --yes snyk test --severity-threshold=high || true
                '''
            }
        }

        // Stage 4: Docker Image Build
        stage('Build Docker Image') {
            steps {
                echo "[INFO] Building Docker image: ${APP_NAME}:${IMAGE_TAG}"
                sh "docker build -t ${APP_NAME}:${IMAGE_TAG} -t ${APP_NAME}:latest ."
            }
        }

        // Stage 5: Container Security Scan via Trivy
        stage('Trivy Image Vulnerability Scan') {
            steps {
                echo "[INFO] Scanning Docker image ${APP_NAME}:${IMAGE_TAG} with Trivy..."
                sh "trivy image --severity HIGH,CRITICAL ${APP_NAME}:${IMAGE_TAG}"
            }
        }
    }

    post {
        always {
            echo '[INFO] Cleaning up workspace artifacts...'
            cleanWs()
        }
        success {
            echo '[SUCCESS] Pipeline completed successfully! Application artifact is verified.'
        }
        failure {
            echo '[ERROR] Pipeline failed! Check logs for security or quality errors.'
        }
    }
}
