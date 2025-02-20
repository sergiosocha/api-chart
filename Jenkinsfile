pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        IMAGE_NAME = 'sergios21/patrones_api'
        IMAGE_TAG = 'v1.0'
        HELM_CHART_DIR = './api_chart'  
    }

    stages {
        stage('Checkout') {
            steps {
                
                git 'https://tu-repositorio-de-git/api-repo.git'
            }
        }

        stage('Compile Project') {
            steps {
                
                sh 'mvn clean install' 
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
                    sh """
                    echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
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

    post {
        always {
            // Limpiar imágenes locales para ahorrar espacio
            sh 'docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true'
        }
        success {
            echo 'El despliegue se realizó correctamente'
        }
        failure {
            echo 'El despliegue falló'
        }
    }
}
