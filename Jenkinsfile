pipeline {
    agent any

    environment {
        IMAGE = "jenkins-practice"
        NAMESPACE = "dev"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:24-alpine'
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        # Use Minikube's Docker daemon
                        eval \$(minikube -p minikube docker-env)
                        
                        # Build image
                        docker build -t ${IMAGE}:${BUILD_NUMBER} .
                        docker tag ${IMAGE}:${BUILD_NUMBER} ${IMAGE}:latest
                        
                        # Verify
                        docker images | grep ${IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh """
                        kubectl config use-context minikube
                        
                        # Create namespace if not exists
                        kubectl create namespace ${NAMESPACE} || true
                        
                        # Apply Kubernetes manifests
                        kubectl apply -f k8s/deployment.yaml -n ${NAMESPACE}
                        kubectl apply -f k8s/service.yaml -n ${NAMESPACE}
                        
                        # Update image
                        kubectl set image deployment/jenkins-practice-app jenkins-practice-app=${IMAGE}:${BUILD_NUMBER} -n ${NAMESPACE}
                        
                        # Wait for rollout
                        kubectl rollout status deployment/jenkins-practice-app -n ${NAMESPACE} --timeout=300s
                        
                        # Show status
                        echo "=== Pods ==="
                        kubectl get pods -n ${NAMESPACE} -l app=jenkins-practice-app
                        
                        echo "=== Services ==="
                        kubectl get svc -n ${NAMESPACE}
                    """
                }
            }
        }
        
        stage('Get Application URL') {
            steps {
                script {
                    sh """
                        echo "========================================="
                        echo "Application Access:"
                        echo "Minikube IP: \$(minikube ip)"
                        minikube service jenkins-practice-app -n ${NAMESPACE} --url
                        echo "========================================="
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "🚀 Deployment completed successfully!"
        }
        failure {
            echo "❌ Build or Deployment Failed!"
            script {
                sh """
                    echo "=== Debug Info ==="
                    kubectl get all -n ${NAMESPACE} || true
                    kubectl describe deployment jenkins-practice-app -n ${NAMESPACE} || true
                """
            }
        }
        always {
            script {
                sh """
                    eval \$(minikube -p minikube docker-env) || true
                    docker image prune -f || true
                """
            }
        }
    }
}