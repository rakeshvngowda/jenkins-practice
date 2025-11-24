pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE = 'rnarasimhaiah/jenkins-practice'
        NAMESPACE = 'dev'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            agent {
                docker {
                    image 'node:24-alpine'
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                sh 'npm ci'
                sh 'NODE_OPTIONS=--experimental-vm-modules npm test'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE}:${BUILD_NUMBER} -t ${IMAGE}:latest .
                    echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push ${IMAGE}:${BUILD_NUMBER}
                    docker push ${IMAGE}:latest
                """
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh """
                    kubectl config use-context minikube
                    kubectl create namespace ${NAMESPACE} || true
                    kubectl set image deployment/jenkins-practice-app jenkins-practice-app=${IMAGE}:${BUILD_NUMBER} -n ${NAMESPACE} || \
                    kubectl apply -f k8s/ -n ${NAMESPACE}
                    kubectl rollout status deployment/jenkins-practice-app -n ${NAMESPACE} --timeout=180s
                """
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline Success!'
            sh 'minikube service jenkins-practice-app -n ${NAMESPACE} --url'
        }
        failure {
            echo '❌ Pipeline Failed!'
        }
        always {
            sh 'docker logout'
        }
    }
}