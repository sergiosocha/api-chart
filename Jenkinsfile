pipeline {
    agent any

    environment {
        HELM_REPO_URL = 'https://sergiosocha.github.io/api-chart-tgz/'  // Repositorio Helm en GitHub Pages
        HELM_CHART = 'api-chart'                                         // Nombre del Chart
        HELM_VERSION = '0.1.0'                                           // Versión del Chart
    }

    stages {
        // 1. Clonar el repositorio de definición del Chart de Helm desde GitHub Pages
        stage('Checkout Helm Chart') {
            steps {
                script {
                    sh """
                    helm repo add api-chart-repo ${HELM_REPO_URL}
                    helm repo update
                    """
                }
            }
        }

        // 2. Desplegar la aplicación con ArgoCD usando el Chart de Helm
        stage('Deploy with ArgoCD') {
            steps {
                script {
                    sh """
                    helm upgrade --install api-release api-chart-repo/${HELM_CHART} --version ${HELM_VERSION}
                    """
                    // Sincronizar con ArgoCD
                    sh """
                    argocd app sync api-release
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finalizado'
        }
        success {
            echo 'Despliegue exitoso'
        }
        failure {
            echo 'Error en el despliegue'
        }
    }
}
