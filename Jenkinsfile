
pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent'
            defaultContainer 'build-tools'
        }
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/avangels-tech/avangelsin-cicd.git', branch: 'main'
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    container('docker') {
                        sh 'docker build -t avangelstech/avangelsin:${BUILD_NUMBER} .'
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push avangelstech/avangelsin:${BUILD_NUMBER}'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh 'sed -i "s/latest/${BUILD_NUMBER}/g" k8s-deployment.yaml'
                    sh 'kubectl apply -f k8s-deployment.yaml'
                }
            }
        }
    }
    post {
    always {
        container('kubectl') {
            sh """
                kubectl delete namespace ${NAMESPACE} || echo "Namespace ${NAMESPACE} not found or already deleted"
            """
        }
        echo "Pipeline completed!"
    }
}
