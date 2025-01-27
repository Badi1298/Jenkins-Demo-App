pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                echo 'Without Docker'
            }
        }

        stage('w/ docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                echo 'With Docker'
                sh 'node --version'
            }
        }
    }
}