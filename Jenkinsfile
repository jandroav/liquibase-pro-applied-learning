pipeline {
    agent any

    stages {
        stage('Status') {
            steps {
                sh 'liquibase status'
            }
        }
    }
}
