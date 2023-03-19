pipeline{
   //agent any
   agent {label 'worker'}
   options{
    buildDiscarder(logRotator(numToKeepStr: '15'))
    disableConcurrentBuilds()
    retry(2)
    timeout(time: 1, unit: 'MINUTES')
   }
   parameters{
    string(name: 'BRANCH', defaultValue: 'master')
    choice(name: 'ENVIRONMENT', choices: ['Dev','QA', 'Prod'])
    booleanParam(name: 'TEST_CASES', defaultValue: false)
   }
   triggers{
    cron('H */4 * * *')
    pollSCM('H */4 * * *')
   }
   stages{
    stage('Build and Push'){
        steps{
            sh '''
            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 234218270377.dkr.ecr.us-east-1.amazonaws.com
            cd vote
            docker build -t 234218270377.dkr.ecr.us-east-1.amazonaws.com/democ41:v${BUILD_NUMBER} .
            docker push 234218270377.dkr.ecr.us-east-1.amazonaws.com/democ41:v${BUILD_NUMBER}
            '''
        }
    }
    stage('Deploy Stage'){
        steps{
            sh '''
            ECR_IMAGE="234218270377.dkr.ecr.us-east-1.amazonaws.com/democ41:v${BUILD_NUMBER}"

            TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition vote-app-c41 --region us-east-1)
            NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')
            NEW_TASK_INFO=$(aws ecs register-task-definition --region us-east-1 --cli-input-json "$NEW_TASK_DEFINTIION")

            NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')
            aws ecs update-service --cluster voteapp --service vote --region us-east-1 --task-definition vote-app-c41:${NEW_REVISION}
            aws ecs describe-services --cluster voteapp --service vote --region us-east-1 --task-definition vote-app-c41:${NEW_REVISION}
            '''
        }
    }
   }
   post{
    always{
        echo "Always Run"
    }
   }
}
