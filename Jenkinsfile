pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        DOCKER_IMAGE = "merajansari87/nodejs-app"
        KUBE_CREDENTIALS_ID = 'kubeconfig'
        GIT_REPO = 'https://github.com/merajansari87/nodejs-app.git'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS_ID) {
                        dockerImage.push("${env.BUILD_NUMBER}")
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: KUBE_CREDENTIALS_ID]) {
                    sh """
                    sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${env.BUILD_NUMBER}|g' deployment.yaml
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    """
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
