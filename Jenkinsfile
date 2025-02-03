pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '784cc9c5-7265-456d-9ef6-1d10d2c7cb69'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
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

        stage ('Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                    sh '''
                        if [ -f build/index.html ]; then
                        echo "index.html found"
                        else
                        echo "index.html not found"
                        exit 1
                        fi
                        npm test
                    '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > netlify-deploy-info.json
                    node_modules/.bin/node-jq -r '.deploy_url' netlify-deploy-info.json
                '''
            }

            post {
                always {
                    script {
                        def deployUrl = readFile('netlify-deploy-info.json').trim()
                        echo "Deployed to: ${deployUrl}"
                        env.CI_ENVIRONMENT_URL = deployUrl
                    }
                }
            }
        }

        stage('Prod E2E Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = '${env.CI_ENVIRONMENT_URL}'
            }

            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }   

        stage('Approval') {
            steps {
                script {
                    def userInput = input(
                        id: 'userInput', message: 'Approve deployment?', parameters: [
                            [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Do you approve the deployment?', name: 'APPROVE']
                        ]
                    )
                    if (!userInput) {
                        error('Deployment not approved. Exiting...')
                    }
                }
            }
        }

        stage('Deploy Production') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod 
                '''
            }
        }

        stage('Prod E2E Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://aesthetic-granita-535577.netlify.app'
            }

            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Production E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }   
    }
}