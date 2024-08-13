def skipRemainingStages = false

pipeline {
    agent any

    stages {
        stage('DEV'){
            when {
              not {
                buildingTag()
              }
            }
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
                tag pattern: "api@\\d.\\d.\\d-rc\$5", comparator: "REGEXP"
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
                tag pattern: "api@\\d.\\d.\\d\$5", comparator: "REGEXP"
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
                tag pattern: "api@\\d.\\d.\\d\$5", comparator: "REGEXP"
            }
            steps {
                script {
                    echo "BUILD AND DEPLOY PROD"
                }
            }
        }
    }
}