pipeline {
    agent any

    tools {
        git 'Default'
    }

    environment {
        NAME           = "python-app"
        VERSION        = "${env.BUILD_ID}-${env.GIT_COMMIT}"
        PROJECT_NAME   = "${JOB_NAME}"
        AWS_ACCOUNT_ID = '485701710361'
        AWS_REGION     = 'eu-north-1'
        ECR_REPO       = 'szgyuval-devops-cicd'
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    def shortSha = sh(
                        returnStdout: true,
                        script: 'git rev-parse --short HEAD'
                    ).trim()
                    env.IMAGE_TAG = "${shortSha}-${env.BUILD_NUMBER}"
                }
                sh '''
                    set -e
                    docker build -t "$ECR_REPO:$IMAGE_TAG" .
                    docker tag "$ECR_REPO:$IMAGE_TAG" "$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG"
                '''
            }
        }

        stage('Login to ECR Repository') {
            steps {
                sh '''
                    aws ecr get-login-password --region "$AWS_REGION" \
                    | docker login --username AWS --password-stdin "$ECR_REGISTRY"
                '''
            }
        }
    }
}
