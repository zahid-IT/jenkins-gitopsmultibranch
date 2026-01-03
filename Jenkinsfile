
pipeline {
    agent any

    environment {
        DOCKER_REPO = "zahidbilal/gitops-jenkins"
        DOCKER_CREDENTIALS = "dockerhub-creds"
        GIT_CREDENTIALS = "github-creds"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', 
                          branches: [[name: "*/${env.BRANCH_NAME}"]],
                          userRemoteConfigs: [[
                              url: "https://github.com/zahid-IT/jenkins-gitopsmultibranch.git",
                              credentialsId: "${GIT_CREDENTIALS}"
                          ]]
                ])
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Use branch name as Docker tag
                    def envTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    echo "Building Docker image for ${env.BRANCH_NAME} with tag ${envTag}"

                    // Build image
                    sh "docker build -t $DOCKER_REPO:$envTag ./app"

                    // Login to DockerHub and push
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", 
                                                      usernameVariable: 'DOCKER_USERNAME', 
                                                      passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push $DOCKER_REPO:$envTag"
                    }

                    // Print info for Image Updater (optional)
                    echo "Image pushed: $DOCKER_REPO:$envTag"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image for ${env.BRANCH_NAME} pushed successfully!"
        }
        failure {
            echo "❌ Docker image build/push failed!"
        }
    }
}
