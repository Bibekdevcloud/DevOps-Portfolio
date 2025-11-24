pipeline {
    agent any

    environment {
        IMAGE_NAME        = 'devops-portfolio'
        IMAGE_TAG         = "${env.BUILD_NUMBER}"
        KIND_CLUSTER_NAME = 'devops-portfolio'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}"
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Load Image into kind') {
            steps {
                sh """
                  echo "Loading image into kind cluster ${KIND_CLUSTER_NAME}"
                  kind load docker-image ${IMAGE_NAME}:${IMAGE_TAG} --name ${KIND_CLUSTER_NAME}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                  echo "Applying Kubernetes manifests (first time or updating)"
                  kubectl apply -f k8s/deployment.yml || true
                  kubectl apply -f k8s/service.yml || true

                  echo "Updating deployment image to ${IMAGE_NAME}:${IMAGE_TAG}"
                  kubectl set image deployment/devops-portfolio \
                    devops-portfolio=${IMAGE_NAME}:${IMAGE_TAG} --record

                  echo "Waiting for rollout to complete..."
                  kubectl rollout status deployment/devops-portfolio
                """
            }
        }
    }

    post {
        always {
            echo "Cleaning up dangling Docker images (if any)..."
            sh 'docker image prune -f || true'
        }
    }
}

