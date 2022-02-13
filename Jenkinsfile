pipeline {
    agent {
      label 'worker'
    }

    options {
            buildDiscarder(logRotator(numToKeepStr: '10'))
            disableConcurrentBuilds()
            timeout(time: 1, unit: 'HOURS')
            timestamps()
    }
    environment {
            AWS_ECR_REGION = 'us-east-1'
            AWS_ECS_SERVICE = 'vote'
            AWS_ECS_TASK_DEFINITION = 'vote-fargate'
            AWS_ECS_COMPATIBILITY = 'FARGATE'
            AWS_ECS_NETWORK_MODE = 'awsvpc'
            AWS_ECS_CPU = '256'
            AWS_ECS_MEMORY = '512'
            AWS_ECS_CLUSTER = 'vote-app'
            AWS_ECS_TASK_DEFINITION_PATH = './ecs/container-definition-update-image.json'
    }
    stages {
      stage('Git Checkout') {
        steps {
          checkout scm
        }
      }
        stage('Build Docker Image') {
            steps {
                sh "eval \$(aws ecr get-login --no-include-email --region us-east-1) && sleep 2"
                sh "aws ecr create-repository --repository-name \${APP_NAME} --region us-east-1 || true"
                sh "cd vote && docker build . -t 635145294553.dkr.ecr.us-east-1.amazonaws.com/tools:\${BUILD_NUMBER}"
                sh "docker push 635145294553.dkr.ecr.us-east-1.amazonaws.com/tools:\${BUILD_NUMBER}"
            }
        }
    }
}
