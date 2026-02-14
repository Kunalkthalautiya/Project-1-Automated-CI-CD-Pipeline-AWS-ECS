pipeline {
  agent any

  environment {
    // AWS Configuration
    AWS_REGION   = 'ap-south-1'
    AWS_ACCOUNT_ID = '<ACCOUNT_ID>'

    // Application Configuration
    APP_NAME     = 'my-app-repo'
    GIT_BRANCH   = 'main'
    GIT_REPO_URL = '<REPO_LINK>'

    // ECS Configuration
    ECS_CLUSTER  = 'my-cluster'
    ECS_SERVICE  = 'my-task-service'

    // Derived / Dynamic Variables
    IMAGE_TAG    = "${BUILD_NUMBER}"
    ECR_REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${APP_NAME}"
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: "${GIT_BRANCH}",
            url: "${GIT_REPO_URL}"
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          docker build -t ${ECR_REPO_URI}:${IMAGE_TAG} .
          docker tag ${ECR_REPO_URI}:${IMAGE_TAG} ${ECR_REPO_URI}:latest
        '''
      }
    }

    stage('Push Image to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region ${AWS_REGION} \
          | docker login --username AWS --password-stdin ${ECR_REPO_URI}

          docker push ${ECR_REPO_URI}:${IMAGE_TAG}
          docker push ${ECR_REPO_URI}:latest
        '''
      }
    }

    stage('Deploy to ECS') {
      steps {
        sh '''
          aws ecs update-service \
            --cluster ${ECS_CLUSTER} \
            --service ${ECS_SERVICE} \
            --force-new-deployment
        '''
      }
    }
  }
}
