pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                sh '''
                    echo 'Without Docker'
                    ls -al
                    touch container-no.txt
                '''
            }
        }

        stage('w/ docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'With Docker'
                    echo 'node --version'
                    ls -al
                    touch container-yes.txt
                '''
            }
        }
    }
}