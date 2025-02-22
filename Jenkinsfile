pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {

                git 'https://github.com/sergiosocha/patrones-back.git'
            }
        }

        stage('Build') {
            steps {

                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {

                sh 'docker build -t sergioss21/patrones-api .'
            }
        }

        stage('Push Docker Image') {
            steps {

                withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u $DOCKER_USER -p $DOCKER_PASSWORD"
                    sh 'docker push sergioss21/patrones-api'
                }
            }
        }

        stage('Update Helm Chart') {
            steps {

                sh """
                yq eval '.image.tag = "latest"' -i helm/patrones-back/values.yaml
                yq eval '.image.repository = "sergioss21/patrones-api"' -i helm/patrones-back/values.yaml
                """
            }
        }

        stage('Package Helm Chart') {
            steps {

                sh 'helm package helm/patrones-back'
            }
        }

        stage('Push Helm Chart') {
            steps {

                withCredentials([usernamePassword(credentialsId: 'HelmRepoCreds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                    curl -u ${USERNAME}:${PASSWORD} --upload-file patrones-back-*.tgz https://github.com/sergiosocha/api-chart-tgz
                    """
                }
            }
        }

        stage('Update ArgoCD Manifest') {
            steps {

                git branch: 'main', credentialsId: 'GitHubCreds', url: 'https://github.com/sergiosocha/api-chart.git'


                sh """
                yq eval '.spec.template.spec.containers[0].image = "sergioss21/patrones-api:latest"' -i deployment.yaml
                """

                sh """
                git add deployment.yaml
                git commit -m "Update Argo manifest with latest image"
                git push origin main
                """
            }
        }
    }

    post {
        always {

            sh 'docker rmi sergioss21/patrones-api'
        }
    }
}
