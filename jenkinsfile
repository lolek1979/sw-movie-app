node {
    // Set up environment variables from credentials and other values
    env.DOCKERHUB_CREDENTIALS = credentials('docker-hub-creds')
    env.GITHUB_CREDENTIALS = credentials('github-creds')
    env.ARGOCD_TOKEN = credentials('argocd-token')
    env.ARGOCD_SERVER = "k8s.orb.local"  // Adjust if needed; this is the address for your Argo CD server via OrbStack
    env.IMAGE_NAME = "pkonieczny321/sw-movie-app"
    env.IMAGE_TAG = "${env.BUILD_NUMBER}"  // Use Jenkins build number for versioning

    stage('Checkout') {
        echo "Checking out sw-movie-app repository..."
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/main']],   // Change 'main' to your branch name if needed
            userRemoteConfigs: [[
                url: 'https://github.com/lolek1979/sw-movie-app.git',
                credentialsId: 'github-creds'
            ]]
        ])
    }

    stage('Build Docker Image') {
        echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}..."
        def image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
    }

    stage('Push Docker Image') {
        echo "Pushing image to Docker Hub..."
        docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDENTIALS}") {
            sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
        }
    }

    stage('Deploy via Argo CD') {
        echo "Deploying sw-movie-app via Argo CD..."
        // Log in to Argo CD (use --insecure if you're using self-signed certificates)
        sh "argocd login ${ARGOCD_SERVER} --auth-token=${ARGOCD_TOKEN} --insecure"
        // Trigger a sync of the sw-movie-app application
        sh "argocd app sync sw-movie-app"
        // Optionally wait for the app to be in a healthy state (adjust timeout if necessary)
        sh "argocd app wait sw-movie-app --sync --health --timeout 300"
    }

    stage('Post Deployment') {
        echo "Deployment triggered successfully. Check Argo CD for the updated sw-movie-app status."
    }
}