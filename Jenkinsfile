import groovy.transform.Field

@Field def skipRemainingStages = false
@Field def CURRENT_ENV = ""
@Field def APP = ""

def boolean checkEnv() {
    def tag = ref
    echo "checking tag $tag"
    if (tag == null) {
        return "dev"
    }
    def parts = tag =~ /(.*)@\d.\d.\d(-rc)?$/
    echo parts.toString()
    def matches = parts.matches()
    if (!matches) {
        return "dev"
    }
    def isStaging = false
    if (parts.size() > 1) {
        isStaging = parts[0][2] == "-rc"
    }

    CURRENT_ENV = isStaging ? "staging": "prod";
    APP = parts[0][1]
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
    parameters {
        string(name: 'ref', defaultValue: 'ref/head/main', description: 'Ref to build')
    }
    stages {
        stage('CHECK TAG') {
            steps {
                checkEnv()
            }
        }
        stage('DEV') {
            when {
                expression {
                    return CURRENT_ENV == "dev"
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
                    return !skipRemainingStages && CURRENT_ENV == "staging"
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
                    return !skipRemainingStages && CURRENT_ENV == "prod"
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
                    return !skipRemainingStages && CURRENT_ENV == "prod"
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