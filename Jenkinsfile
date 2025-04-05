node {
    // Run inside a Docker container that has the Docker CLI.
    docker.image('docker:20.10.7').inside('-v /var/run/docker.sock:/var/run/docker.sock') {
        // Define image details. You can use a fixed tag like "1.0.0" or incorporate BUILD_NUMBER.
        def imageName = "pkonieczny321/sw-movie-app"
        def imageTag = "1.0.0" // For example; alternatively, use "${env.BUILD_NUMBER}" or a combination.

        stage('Checkout') {
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

        stage('Build Docker Image') {
            echo "Building Docker image ${imageName}:${imageTag}..."
            sh "docker build -t ${imageName}:${imageTag} ."
        }

        stage('Push Docker Image') {
            echo "Logging in to Docker Hub and pushing the image..."
            // Use withCredentials to access Docker Hub credentials.
            withCredentials([usernamePassword(
                credentialsId: 'docker-hub-creds',
                usernameVariable: 'DOCKERHUB_USER',
                passwordVariable: 'DOCKERHUB_PASSWORD'
            )]) {
                sh "docker login -u ${DOCKERHUB_USER} -p ${DOCKERHUB_PASSWORD}"
            }
            sh "docker push ${imageName}:${imageTag}"
        }

        stage('Deploy via Argo CD') {
            echo "Deploying sw-movie-app via Argo CD..."
            // Use withCredentials for the Argo CD token.
            withCredentials([string(credentialsId: 'argocd-token', variable: 'ARGOCD_TOKEN')]) {
                // Log in to Argo CD. Adjust the server address if needed.
                sh "argocd login k8s.orb.local --auth-token=${ARGOCD_TOKEN} --insecure"
                // Trigger a sync of the sw-movie-app application.
                sh "argocd app sync sw-movie-app"
                // Wait for the application to become synced and healthy.
                sh "argocd app wait sw-movie-app --sync --health --timeout 300"
            }
        }

        stage('Post Deployment') {
            echo "Deployment triggered successfully. Check Argo CD for the updated sw-movie-app status."
        }
    } // End of docker.image block.
}