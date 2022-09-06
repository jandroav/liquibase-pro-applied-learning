pipeline {
    agent {label 'mac'}

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
