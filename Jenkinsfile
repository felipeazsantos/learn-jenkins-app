pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
    }

    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.27.50'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh ''' 
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-denifition-prod.json
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
