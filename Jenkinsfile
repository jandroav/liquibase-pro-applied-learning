pipeline {
    agent {label 'mac'}

    stages {
        stage('Status') {
            steps {
                sh 'liquibase status'
            }
        }
        stage('Check SQL') {
            steps {
                sh 'liquibase update-sql'
            }
        }
    }
}
