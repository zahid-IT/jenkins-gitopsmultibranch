
pipeline {
    agent any

    environment {
        DOCKER_REPO = "zahidbilal/gitops-jenkins"
        DOCKER_CREDENTIALS = "dockerhub-creds" // ID of Jenkins Docker credentials
    }

    stages {
        stage('Build & Push Docker Image') {
            steps {
                script {
                    def envTag = "${env.BRANCH_NAME}-latest"
                    echo "Building Docker image for ${env.BRANCH_NAME}"
                    sh "docker build -t $DOCKER_REPO:$envTag ./app"

                    // Fixed Docker login using Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS,
                                                     usernameVariable: 'DOCKER_USERNAME',
                                                     passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push $DOCKER_REPO:$envTag"
                    }
                }
            }
        }

        stage('Helm Deploy') {
            steps {
                script {
                    def valuesFile = "helm-chart/nginx-app/values/${env.BRANCH_NAME}-values.yaml"
                    sh "helm upgrade --install nginx-app helm-chart/nginx-app -f $valuesFile --namespace ${env.BRANCH_NAME} --create-namespace"
                }
            }
        }

        stage('ArgoCD Sync') {
            steps {
                script {
                    sh "argocd app sync nginx-${env.BRANCH_NAME}"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment for ${env.BRANCH_NAME} successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
