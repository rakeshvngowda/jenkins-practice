pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "jenkins-practice-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        MINIKUBE_IP = sh(script: 'minikube ip', returnStdout: true).trim()
        KUBECONFIG = "${HOME}/.kube/config"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh """
                        eval \$(minikube docker-env)
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh '''
                    echo "Add your test commands here"
                    # npm test or python -m pytest, etc.
                '''
            }
        }
        
        stage('Push to Minikube Registry') {
            steps {
                script {
                    echo "Image available in Minikube Docker daemon"
                    sh """
                        eval \$(minikube docker-env)
                        docker images | grep ${DOCKER_IMAGE}
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Minikube...'
                    sh """
                        kubectl config use-context minikube
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        kubectl set image deployment/jenkins-practice-app jenkins-practice-app=${DOCKER_IMAGE}:${DOCKER_TAG} --record
                        kubectl rollout status deployment/jenkins-practice-app
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    echo 'Verifying deployment...'
                    sh """
                        kubectl get pods -l app=jenkins-practice-app
                        kubectl get svc jenkins-practice-app
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
            sh """
                echo "Application URL: http://\$(minikube ip):\$(kubectl get svc jenkins-practice-app -o jsonpath='{.spec.ports[0].nodePort}')"
            """
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Cleaning up...'
            sh '''
                docker image prune -f
            '''
        }
    }
}
