pipeline {
    agent any

    stages {
        stage('Status') {
            steps {
                sh 'pwd'
                sh 'which liquibase'
                sh 'liquibase status'
            }
        }
    }
}
