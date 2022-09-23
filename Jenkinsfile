pipeline {

    parameters {
        string(name: 'rollback_to_tag', defaultValue: '', description: 'Specify the tag to rollback to')
    }

    environment {
        POSTGRESQL_STORE_CREDS = credentials('aws-enterprisedb')
        POSTGRESQL_URL = "${env.POSTGRESQL_URL}"
        POSTGRESQL_REF_URL = "${env.POSTGRESQL_REF_URL}"
    }

    agent {label 'linux'}

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
                sh 'liquibase --changelog-file=./changelogs/dbchangelog.xml --url=' + POSTGRESQL_URL + ' --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --liquibase-schema-name=public status'
            }
        }
        stage('Check SQL') {
            when {
                expression { params.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase --changelog-file=./changelogs/dbchangelog.xml --url=' + POSTGRESQL_URL + ' --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --liquibase-schema-name=public update-sql'
            }
        }
        stage('Deploy changetsets') {
            when {
                expression { params.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase --changelog-file=./changelogs/dbchangelog.xml --url=' + POSTGRESQL_URL + ' --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --liquibase-schema-name=public --hub-mode=off update'
            }
        }
        stage('Rollback to tag') {
            when {
                expression { !params.rollback_to_tag.isEmpty() }
            }
            steps {
                echo 'Trying to rollback to ' + params.rollback_to_tag + ' tag'
                sh 'liquibase --changelog-file=./changelogs/dbchangelog.xml --url=' + POSTGRESQL_URL + ' --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --liquibase-schema-name=public --hub-mode=off rollback ' + params.rollback_to_tag
            }
        }
        stage('Diff against prod') {
            steps {
                sh 'liquibase --changelog-file=prod_diff.xml --url=' + POSTGRESQL_URL + ' --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --referenceUsername=' + POSTGRESQL_STORE_CREDS_USR + ' --referencePassword=' + POSTGRESQL_STORE_CREDS_PSW + ' --referenceUrl=' + POSTGRESQL_REF_URL + ' --liquibase-schema-name=public diff'
                sh 'liquibase --changelog-file=prod_diff.xml --url=' + POSTGRESQL_URL + ' --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --referenceUsername=' + POSTGRESQL_STORE_CREDS_USR + ' --referencePassword=' + POSTGRESQL_STORE_CREDS_PSW + ' --referenceUrl=' + POSTGRESQL_REF_URL + ' --liquibase-schema-name=public diff-changelog'
                archiveArtifacts artifacts: 'prod_diff.xml', fingerprint: true
            }
        }
    }
}
