pipeline {

    parameters {
        string(name: 'rollback-to-tag', defaultValue: '', description: 'Specify the tag to rollback to')
    }

    agent {label 'mac'}

    stages {
        stage('Check input parameters') {
            steps {
                script {
                    parameters.each { param ->
                        echo "${param.key} -> ${param.value} "
                    }
                }
            }
        }
        stage('Status') {
            steps {
                sh 'liquibase status'
            }
        }
        stage('Check SQL') {
            when {
                expression { parameters.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase update-sql'
            }
        }
        stage('Deploy changetsets') {
            when {
                expression { parameters.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase update'
            }
        }
        stage('Rollback to tag') {
            when {
                expression { !parameters.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase rollback ${parameters.rollback_to_tag}'
            }
        }
    }
}
