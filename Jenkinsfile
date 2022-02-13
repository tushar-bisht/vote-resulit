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
            AWS_DEFAULT_REGION = 'us-east-1'
            AWS_ECS_SERVICE = 'vote'
            TASK_FAMILY = 'vote-fargate'
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
        stage('Deploy in ECS') {
            steps {

                script {
                    ECR_IMAGE="635145294553.dkr.ecr.us-east-1.amazonaws.com/tools:${BUILD_NUMBER}"
                    TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition "$TASK_FAMILY" --region "$AWS_DEFAULT_REGION")
                    NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities)')
                    NEW_TASK_INFO=$(aws ecs register-task-definition --region "$AWS_DEFAULT_REGION" --cli-input-json "$NEW_TASK_DEFINTIION")
                    NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')
                    aws ecs update-service --cluster ${ECS_CLUSTER} \
                                        --service ${SERVICE_NAME} \
                                        --task-definition ${TASK_FAMILY}:${NEW_REVISION}
                }
            }
        }
    }

post {
        always {
            deleteDir()
            sh "docker rmi 635145294553.dkr.ecr.us-east-1.amazonaws.com/tools:\${BUILD_NUMBER}"
            }
        }
}
}
