pipeline {
    agent any

    tools {
        git 'Default'
    }

    environment {
      NAME = "python-app"
      VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
      IMAGE_REPO = "szgyuval123/gitops-repo"
      PROJECT_NAME = "${JOB_NAME}"
    }

    stages {
        stage('Build Image') {
            steps {
                sh "docker build -t ${NAME} ."
                sh "docker tag ${NAME}:latest ${IMAGE_REPO}:${NAME}-${VERSION}"
            }
        }

        stage('Tests') {
            steps {
                echo "Tests should be added!"
            }
        }

        stage('Push Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-hub-creds', url: '') {
                    sh  "docker image push ${IMAGE_REPO}:${NAME}-${VERSION}"
                }
            }
        }

    }
}