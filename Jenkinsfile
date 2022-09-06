pipeline {

    parameters {
        string(name: 'rollback_to_tag', defaultValue: '', description: 'Specify the tag to rollback to')
    }

    agent {label 'mac'}

    stages {
        stage('Check input parameters') {
            steps {
                script {
                    params.each { param ->
                        echo "${param.key} -> ${param.value}"
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
                expression { params.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase update-sql'
            }
        }
        stage('Deploy changetsets') {
            when {
                expression { params.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase update'
            }
        }
        stage('Rollback to tag') {
            when {
                expression { !params.rollback_to_tag.isEmpty() }
            }
            steps {
                echo 'Trying to rollback to ' + params.rollback_to_tag + ' tag'
                sh 'liquibase rollback ' + params.rollback_to_tag
            }
        }
        stage('Diff against prod') {
            steps {
                sh 'liquibase diff'
                sh 'liquibase --changelog-file=prod_diff.xml diff-changelog'
                archiveArtifacts artifacts: 'prod_diff.xml', fingerprint: true
            }
        }
    }
}
