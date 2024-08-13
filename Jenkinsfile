def skipRemainingStages = false

pipeline {
    agent any

    stages {
        stage('DEV'){
            steps {
                script {
                    skipRemainingStages = true
                    echo "BUILD AND DEPLOY DEV"
                }
            }
        }
        stage('STAGING') {
            when {
                expression {
                    !skipRemainingStages
                }
            }
            steps {
                script {
                    skipRemainingStages = true
                    echo "BUILD AND DEPLOY STAGING"
                }
            }
        }
        stage('PROD APPROVAL') {
            when {
                expression {
                    !skipRemainingStages
                }
            }
            steps {
                script {
                    def userInput = input(id: 'confirm', message: 'Approve deployment to PROD?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you agree with this'] ])
                    if(!userInput) {
                        error('Deployment to PROD was not approved')
                    }
                }
            }
        }
        stage('PROD') {
            when {
                expression {
                    !skipRemainingStages
                }
            }
            steps {
                script {
                    echo "BUILD AND DEPLOY PROD"
                }
            }
        }
    }
}