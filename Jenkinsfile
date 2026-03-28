@Library('jenkins-shared') _

pipeline {
    agent any
    environment {
        DOCKER_REPO = "hiteshnikam/smart-manufacturing-mlops"
    }
    stages {
        stage('Checkout') {
            steps {
                gitCheckout('https://github.com/hiteshnikam121/MLOPS-Jenkins-Shared-Library-CI-CD.git','*/main','github-token')

            }
        }

        stage('Build & Push Image') {
            steps {
                dockerBuildAndPush(DOCKER_REPO,'dockerhub-token')
            }
        }

        stage('Install Kubectl') {
            steps {
               installKubectl()
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                k8sDeploy('kubeconfig')
            }
        }
    }
}
