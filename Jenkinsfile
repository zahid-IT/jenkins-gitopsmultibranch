
pipeline {
    agent {
        kubernetes {
            label 'docker-agent'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24.0.5
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }

    environment {
        DOCKER_REPO = "zahidbilal/gitops-jenkins"
        DOCKER_CREDENTIALS = "dockerhub-creds"
        GIT_CREDENTIALS = "github-creds"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                container('docker') {
                    script {
                        def envTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        sh "docker build -t $DOCKER_REPO:$envTag ./app"

                        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}",
                                                          usernameVariable: 'DOCKER_USERNAME',
                                                          passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                            sh "docker push $DOCKER_REPO:$envTag"
                        }
                    }
                }
            }
        }
    }
}
