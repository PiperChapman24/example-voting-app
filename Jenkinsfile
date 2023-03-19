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
            docker build -t 234218270377.dkr.ecr.us-east-1.amazonaws.com/democ41:v${BUILD_NUMBER} .
            docker push 234218270377.dkr.ecr.us-east-1.amazonaws.com/democ41:v${BUILD_NUMBER}
            '''
        }
    }
    stage('Deploy Stage'){
        steps{
            sh "sleep 10"
        }
    }
   }
   post{
    always{
        echo "Always Run"
    }
   }
}
