
pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent'
            yamlFile 'kaniko-pod-template.yaml'
        }
    }

    environment {
        DOCKER_REPO = "zahidbilal/gitops-jenkins"
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
                container('kaniko') {
                    script {
                        def imageTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
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
}
