
pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  volumes:
  - name: kaniko-secret
    secret:
      secretName: dockerhub-creds # Kubernetes secret with Docker credentials
"""
        }
    }

    environment {
        DOCKER_REPO = "zahidbilal/gitops-jenkins"
        GIT_CREDENTIALS = "github-creds"
        // Note: BRANCH_NAME is provided by multibranch pipeline
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
                container('kaniko') {
                    script {
                        // Create unique tag for branch + build number
                        def imageTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        echo "Building image: $DOCKER_REPO:$imageTag"

                        sh """
                        /kaniko/executor \
                            --context \$WORKSPACE/app \
                            --dockerfile \$WORKSPACE/app/Dockerfile \
                            --destination $DOCKER_REPO:$imageTag \
                            --destination $DOCKER_REPO:latest \
                            --verbosity info
                        """
                    }
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
