pipeline {
    agent {
        docker {
            image 'docker:20.10.7'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    parameters {
        string(name: 'BASE_VERSION', defaultValue: '1.0', description: 'Set the base version (e.g., 2.0)')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-creds')
        GITHUB_CREDENTIALS = credentials('github-creds')
        ARGOCD_TOKEN = credentials('argocd-token')
        ARGOCD_SERVER = "k8s.orb.local"
        IMAGE_NAME = "pkonieczny321/sw-movie-app"
        // Combine the parameter with the Jenkins BUILD_NUMBER to create a semantic version tag
        IMAGE_TAG = "${params.BASE_VERSION}.${env.BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out sw-movie-app repository..."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/lolek1979/sw-movie-app.git',
                        credentialsId: 'github-creds'
                    ]]
                ])
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        stage('Push Docker Image') {
            steps {
                echo "Pushing image to Docker Hub..."
                sh "docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}"
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
        stage('Deploy via Argo CD') {
            steps {
                echo "Deploying sw-movie-app via Argo CD..."
                sh "argocd login ${ARGOCD_SERVER} --auth-token=${ARGOCD_TOKEN} --insecure"
                sh "argocd app sync sw-movie-app"
                sh "argocd app wait sw-movie-app --sync --health --timeout 300"
            }
        }
    }
    post {
        success {
            echo "Deployment succeeded!"
        }
        failure {
            echo "Deployment failed. Check logs for errors."
        }
    }
}