
stage('Build & Push Docker Image') {
    steps {
        script {
            def envTag = "${env.BRANCH_NAME}-latest"
            echo "Building Docker image for ${env.BRANCH_NAME}"
            sh "docker build -t $DOCKER_REPO:$envTag ./app"

            // Correct Docker login using Jenkins credentials
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                                             usernameVariable: 'DOCKER_USERNAME', 
                                             passwordVariable: 'DOCKER_PASSWORD')]) {
                sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                sh "docker push $DOCKER_REPO:$envTag"
            }
        }
    }
}
