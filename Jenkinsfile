def skipRemainingStages = false
def currentEnv = "dev"

def String readCurrentTag() {
    return sh(returnStdout: true, script: "git describe --tags").trim()
}

def boolean checkEnv() {
    def tag = readCurrentTag()
    echo "checking tag $tag"
    if (tag == null) {
        return "dev"
    }
    def isStaging = tag =~ /api@\d.\d.\d-rc$/
    def isProd = tag =~ /api@\d.\d.\d$/
    return isStaging.matches() ? "staging": isProd.matches() ? "prod": "dev"
}

pipeline {
    agent any
    stages {
        stage('CHECK TAG') {
            steps {
                script {
                    currentEnv = checkEnv()
                }
            }
        }
        stage('DEV') {
            when {
                expression {
                    return currentEnv == "dev"
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
                    return !skipRemainingStages && currentEnv == "staging"
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
                    return !skipRemainingStages && currentEnv == "prod"
                }
            }
            steps {
                script {
                    def userInput = input(id: 'confirm', message: 'Approve deployment to PROD?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you agree with this']])
                    if (!userInput) {
                        error('Deployment to PROD was not approved')
                    }
                }
            }
        }
        stage('PROD') {
            when {
                expression {
                    return !skipRemainingStages && currentEnv == "prod"
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