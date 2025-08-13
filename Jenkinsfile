pipeline {
    agent any

    tools {
        git 'Default'
    }

    environment {
        NAME           = "python-app"
        PROJECT_NAME   = "${JOB_NAME}"
        AWS_ACCOUNT_ID = '485701710361'
        AWS_REGION     = 'eu-north-1'
        ECR_REPO       = 'szgyuval-devops-cicd'
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                // Ensure VERSION file exists (defaults to 1.0.0 on first run)
                sh 'test -f VERSION || echo "1.0.0" > VERSION'
            }
        }

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

        stage('Push Image') {
            steps {
                sh '''
                set -e
                # Push versioned tag
                docker push "$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG"
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    # If the container exists, stop and remove it
                    if [ "$(docker ps -a -q -f name=python-app)" ]; then
                        echo "Container 'python-app' already exists. Removing..."
                        docker rm -f python-app
                    fi

                    # Run new container from latest image
                    docker run -d -p 5000:5000 --name python-app "$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG"
                    '''
            }
        }
    }
}
