import groovy.transform.Field

@Field def skipRemainingStages = false
@Field def CURRENT_ENV = ""
@Field def APP = ""

def String readCurrentTag() {
    return sh(returnStdout: true, script: "git describe --tags").trim()
}

def boolean checkEnv() {
    def tag = readCurrentTag()
    echo "checking tag $tag"
    if (tag == null) {
        return "dev"
    }
    def check = tag =~ /(.*)@\d.\d.\d(-rc)?$/
    def matches = check.matches()
    if (matches) {
        return "dev"
    }
    def isStaging = false
    if (matches.size() > 1) {
        isStaging = matches[0][2] == "-rc"
    }

    CURRENT_ENV = isStaging ? "staging": isProd ? "prod": "dev"
    APP = matches[0][1]
}

def build(app) {
    if (currentEnv == "staging" || currentEnv == "prod") {
        if (app != APP) {
            return
        }
        return
    }

    echo "Building $app"
}

pipeline {
    agent any
    stages {
        stage('CHECK TAG') {
            steps {
                checkEnv()
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
                    build("api")
                    build("queue")
                    build("portal")
                    build("landing")
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
                    build("api")
                    build("queue")
                    build("portal")
                    build("landing")
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
                    build("api")
                    build("queue")
                    build("portal")
                    build("landing")
                }
            }
        }
    }
}