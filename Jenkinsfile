def skipRemainingStages = false
def envvvvv = "dev"

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
    def result = isStaging.matches() ? "staging" : isProd.matches() ? "prod" : "dev"
    echo "env: $result"
    return result
}

def tess() {
  echo envvvvv == "dev"
  echo envvvvv == "staging"
  echo envvvvv == "prod"
}

pipeline {
    agent any

    stages {
        stage('Check Tag') {
            steps {
                script {
                  envvvvv = checkEnv()
                  tess()
                }
            }
        }
        stage('DEV'){
            when {
              expression {
                envvvvv == "dev"
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
                   !skipRemainingStages && envvvvv == "staging"
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
                  !skipRemainingStages && envvvvv == "prod"
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
                    !skipRemainingStages && envvvvv == "prod"
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