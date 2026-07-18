pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  parameters {
    choice(
      name: 'AWS_REGION',
      choices: ['ap-south-1'],
      description: 'AWS region for ECR'
    )
  }

  stages {
    stage('Initialize AWS configuration') {
      steps {
        script {
          env.AWS_REGION = params.AWS_REGION

          env.AWS_ACCOUNT_ID = sh(
            script: 'aws sts get-caller-identity --query Account --output text',
            returnStdout: true
          ).trim()

          env.ECR_REGISTRY =
            "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com"

          env.IMAGE_TAG = "jenkins-${env.BUILD_NUMBER}"
        }

        echo "Using AWS region: ${env.AWS_REGION}"
        echo "Using ECR registry: ${env.ECR_REGISTRY}"
        echo "Using image tag: ${env.IMAGE_TAG}"
      }
    }

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Preflight') {
      steps {
        sh '''
          docker version --format '{{.Server.Version}}'
          aws sts get-caller-identity --region "$AWS_REGION"
        '''
      }
    }

    stage('Login to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region "$AWS_REGION" | \
          docker login --username AWS --password-stdin "$ECR_REGISTRY"
        '''
      }
    }

    stage('Build images') {
      steps {
        sh '''
          docker build --platform linux/amd64 \
            --build-arg REACT_APP_AUTH_API_URL=/api \
            --build-arg REACT_APP_STREAMING_API_URL=/api \
            --build-arg REACT_APP_STREAMING_PUBLIC_URL= \
            --build-arg REACT_APP_ADMIN_API_URL=/api/admin \
            --build-arg REACT_APP_CHAT_API_URL=/api/chat \
            --build-arg REACT_APP_CHAT_SOCKET_URL= \
            -t "$ECR_REGISTRY/streaming-frontend:$IMAGE_TAG" \
            ./StreamingApp/frontend

          docker build --platform linux/amd64 \
            -t "$ECR_REGISTRY/streaming-auth:$IMAGE_TAG" \
            ./StreamingApp/backend/authService

          docker build --platform linux/amd64 \
            -f ./StreamingApp/backend/streamingService/Dockerfile \
            -t "$ECR_REGISTRY/streaming-streaming:$IMAGE_TAG" \
            ./StreamingApp/backend

          docker build --platform linux/amd64 \
            -f ./StreamingApp/backend/adminService/Dockerfile \
            -t "$ECR_REGISTRY/streaming-admin:$IMAGE_TAG" \
            ./StreamingApp/backend

          docker build --platform linux/amd64 \
            -f ./StreamingApp/backend/chatService/Dockerfile \
            -t "$ECR_REGISTRY/streaming-chat:$IMAGE_TAG" \
            ./StreamingApp/backend
        '''
      }
    }

    stage('Push images to ECR') {
      steps {
        sh '''
          docker push "$ECR_REGISTRY/streaming-frontend:$IMAGE_TAG"
          docker push "$ECR_REGISTRY/streaming-auth:$IMAGE_TAG"
          docker push "$ECR_REGISTRY/streaming-streaming:$IMAGE_TAG"
          docker push "$ECR_REGISTRY/streaming-admin:$IMAGE_TAG"
          docker push "$ECR_REGISTRY/streaming-chat:$IMAGE_TAG"
        '''
      }
    }
  }

  post {
    success {
      echo "Successfully pushed all images with tag: ${env.IMAGE_TAG}"
    }

    failure {
      echo "Pipeline failed. Check the failed stage console output."
    }
  }
}