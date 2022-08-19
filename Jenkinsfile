pipeline {
  agent {
    label 'slave'
  }
  stages {
    stage('Git Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      parallel {
        stage('Build Docker Image') {
          steps {
            sh 'cd vote && sudo docker build . -t 776487083560.dkr.ecr.us-east-1.amazonaws.com/tushar:${BUILD_NUMBER}'
            sh 'sudo docker push 776487083560.dkr.ecr.us-east-1.amazonaws.com/tushar:${BUILD_NUMBER}'
          }
        }

        stage('Unit Testing') {
          steps {
            sh 'echo Run the Test Cases'
          }
        }

      }
    }

    stage('Deploy in ECS') {
      steps {
        script {
          sh'''
ECR_IMAGE="776487083560.dkr.ecr.us-east-1.amazonaws.com/tushar:${BUILD_NUMBER}"
TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition "$TASK_FAMILY" --region "$AWS_DEFAULT_REGION")
NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')
NEW_TASK_INFO=$(aws ecs register-task-definition --region "$AWS_DEFAULT_REGION" --cli-input-json "$NEW_TASK_DEFINTIION")
NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')
aws ecs update-service --cluster ${ECS_CLUSTER} \
--service ${SERVICE_NAME} \
--task-definition ${TASK_FAMILY}:${NEW_REVISION}'''
        }

      }
    }

    stage('Adding New Stage') {
      steps {
        sh 'echo adding a new stage'
      }
    }

  }
  environment {
    AWS_DEFAULT_REGION = 'us-east-1'
    SERVICE_NAME = 'voteapp'
    TASK_FAMILY = 'voteapp'
    ECS_CLUSTER = 'vote'
  }
  post {
    always {
      deleteDir()
      sh 'sudo docker rmi 776487083560.dkr.ecr.us-east-1.amazonaws.com/tushar:${BUILD_NUMBER}'
    }

  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }
}
