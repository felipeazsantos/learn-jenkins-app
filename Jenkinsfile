pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
        AWS_ECS_CLUSTER= 'LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_SERVICE= 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TASK= 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.27.50'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh ''' 
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service  --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                    '''
                }
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

    
    }
}
