pipeline {
    agent any

    environment {
        IMAGE_NAME = 'sergios21/api-patrones-chart'
        IMAGE_TAG = 'v1.0'
        HELM_CHART_DIR = './api_chart'  
    }

    stages {
         stage('Checkout') {
            steps {
                git branch: 'main', 'https://github.com/sergiosocha/api-chart.git'
            }
        }


        stage('Build Docker Image') {
            steps {
                
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                 script {
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USER -p $DOCKER_PASSWORD"
                        sh 'docker push sergioss21/spring-api'
                    }
                }
            }
        }

        stage('Update Helm Chart') {
            steps {
                
                script {
                    sh """
                    sed -i 's/tag:.*/tag: "${IMAGE_TAG}"/' ${HELM_CHART_DIR}/values.yaml
                    """
                }
            }
        }

        stage('Deploy with ArgoCD') {
            steps {
                
                script {
                    sh """
                    argocd app sync api-release
                    """
                }
            }
        }
    }
}
