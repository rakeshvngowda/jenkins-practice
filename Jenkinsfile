pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build') {
            steps { echo "Building branch: ${env.BRANCH_NAME}" }
        }
        stage('Test') {
            steps { echo "Running tests..." }
        }
    }
    post {
        always {
            echo "Status: ${currentBuild.currentResult}"
        }
    }
}
