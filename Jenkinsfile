pipeline {

    parameters {
        string(name: 'rollback_to_tag', defaultValue: '', description: 'Specify the tag to rollback to')
    }

    environment {
        POSTGRESQL_STORE_CREDS = credentials('postgresql-store')
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
                sh 'liquibase --changelog-file=./changelogs/dbchangelog.xml --url=jdbc:postgresql://localhost:5432/Store --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' status'
            }
        }
        stage('Check SQL') {
            when {
                expression { params.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase --changelog-file=./changelogs/dbchangelog.xml --url=jdbc:postgresql://localhost:5432/Store --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' update-sql'
            }
        }
        stage('Deploy changetsets') {
            when {
                expression { params.rollback_to_tag.isEmpty() }
            }
            steps {
                sh 'liquibase --changelog-file=./changelogs/dbchangelog.xml --url=jdbc:postgresql://localhost:5432/Store --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --hub-mode=off update'
            }
        }
        stage('Rollback to tag') {
            when {
                expression { !params.rollback_to_tag.isEmpty() }
            }
            steps {
                echo 'Trying to rollback to ' + params.rollback_to_tag + ' tag'
                sh 'liquibase --changelog-file=./changelogs/dbchangelog.xml --url=jdbc:postgresql://localhost:5432/Store --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --hub-mode=off rollback ' + params.rollback_to_tag
            }
        }
        stage('Diff against prod') {
            steps {
                sh 'liquibase --changelog-file=prod_diff.xml --url=jdbc:postgresql://localhost:5432/Store --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --referenceUsername=' + POSTGRESQL_STORE_CREDS_USR + ' --referencePassword=' + POSTGRESQL_STORE_CREDS_PSW + ' --referenceUrl=jdbc:postgresql://localhost:5432/store_prod diff'
                sh 'liquibase --changelog-file=prod_diff.xml --url=jdbc:postgresql://localhost:5432/Store --username=' + POSTGRESQL_STORE_CREDS_USR + ' --password=' + POSTGRESQL_STORE_CREDS_PSW + ' --referenceUsername=' + POSTGRESQL_STORE_CREDS_USR + ' --referencePassword=' + POSTGRESQL_STORE_CREDS_PSW + ' --referenceUrl=jdbc:postgresql://localhost:5432/store_prod diff-changelog'
                archiveArtifacts artifacts: 'prod_diff.xml', fingerprint: true
            }
        }
    }
}
