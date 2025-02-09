pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '784cc9c5-7265-456d-9ef6-1d10d2c7cb69'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
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

        stage('Deploy to AWS ECS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args '--entrypoint=""'
                    reuseNode true
                }
            }
            environment {
                AWS_CLUSTER_NAME = 'Jenkins-Cluster-Prod'
                AWS_SERVICE_NAME = 'LearnJenkinsApp-Service-Prod '
                TASK_DEFINITION_FILE = 'aws/task-definition-prod.json'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-cli', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://$TASK_DEFINITION_FILE
                        aws ecs update-service --cluster $AWS_CLUSTER_NAME --service $AWS_SERVICE_NAME --force-new-deployment
                    '''
                }
            }
        }
    }
}