pipeline {
    agent any

    environment {
        APP_NAME = 'learningjenkinsapp'
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
        AWS_DEFAULT_REGION = 'us-east-2'
        AWS_DOCKER_REGISTRY = '663375301870.dkr.ecr.us-east-2.amazonaws.com'
        AWS_ECS_CLUSTER= 'LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_SERVICE_PROD= 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TASK_PROD= 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {
        stage('Build App') {
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
        
        stage('Build Docker Image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }

            steps {
                 withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        amazon-linux-extras install docker
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                 }
            }
        }

        stage('AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
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
                        aws ecs update-service  --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TASK_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
            }
        }

    
    }
}
