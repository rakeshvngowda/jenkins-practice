pipeline {
    agent any

    environment {
        IMAGE = "jenkins-practice"
        REGISTRY = "localhost:5000"
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
                sh """
                    docker build -t ${IMAGE}:latest .
                """
            }
        }
        
        stage('Push Image to Local Registry') {
            steps {
                sh "docker push ${REGISTRY}/${IMAGE}:latest"
            }
        }

        stage('Deploy to Minikube') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG_PATH')]) {
                    sh """
                        export KUBECONFIG=$KUBECONFIG_PATH
                        kubectl config use-context minikube
                        kubectl apply -f k8s/deployment.yaml -n dev --validate=false
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
        }
    }
}