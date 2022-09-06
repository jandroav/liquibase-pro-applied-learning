pipeline {

    parameters {
        string(name: 'rollback-to-tag', defaultValue: '', description: 'Specify the tag to rollback to')
    }

    agent {label 'mac'}

    stages {
        stage('Status') {
            steps {
                sh 'liquibase status'
            }
        }
        stage('Check SQL') {
            when {
                expression { null == params.rollback_to_tag }
            }
            steps {
                sh 'liquibase update-sql'
            }
        }
        stage('Deploy changetsets') {
            when {
                expression { null == params.rollback_to_tag }
            }
            steps {
                sh 'liquibase update'
            }
        }
        stage('Rollback to tag') {
            when {
                expression { null != params.rollback_to_tag }
            }
            steps {
                sh 'liquibase rollback ${params.rollback_to_tag}'
            }
        }
    }
}
