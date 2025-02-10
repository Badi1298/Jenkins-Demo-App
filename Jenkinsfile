pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = "myjenkinsapp"
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Building the project"
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
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    amazon-linux-extras install docker
                    docker build -t $APP_NAME:$REACT_APP_VERSION .
                '''
            }
        }

        stage('Deploy to AWS ECS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    args '--entrypoint=""'
                    reuseNode true
                }
            }
            environment {
                AWS_CLUSTER_NAME = 'Jenkins-Cluster-Prod'
                AWS_SERVICE_NAME = 'LearnJenkinsApp-Service-Prod'
                AWS_TASK_NAME = 'LearnJenkinsApp-TaskDefinition-Prod'
                TASK_DEFINITION_FILE = 'aws/task-definition-prod.json'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-cli', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install -y jq
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://$TASK_DEFINITION_FILE | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster $AWS_CLUSTER_NAME --service $AWS_SERVICE_NAME --task-definition $AWS_TASK_NAME:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_CLUSTER_NAME --services $AWS_SERVICE_NAME
                    '''
                }
            }
        }
    }
}