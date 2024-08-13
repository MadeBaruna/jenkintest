import groovy.transform.Field

@Field def skipRemainingStages = false
@Field def CURRENT_ENV = "dev"
@Field def APP = ""

def boolean checkEnv() {
    def tag = ref.replaceAll(/ref\/.*\//, "").trim()
    
    echo "checking tag $tag"
    if (tag == "main") {
        CURRENT_ENV = "dev";
        return
    }
    def parts = tag =~ /(.*)@\d.\d.\d(-rc)?$/
    def matches = parts.matches()
    if (!matches) {
        return "dev"
    }
    def isStaging = parts[0][2] == "-rc"

    CURRENT_ENV = isStaging ? "staging": "prod";
    APP = parts[0][1]

    echo "Current env: $CURRENT_ENV"
    echo "App: $APP"
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
            stages {
                stage('API') {
                    steps {
                        echo "BUILD API"
                    }
                }
                stage('QUEUE') {
                    steps {
                        echo "BUILD QUEUE"
                    }
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
                echo "BUILD STAGING"
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
                echo "BUILD PROD"
            }
        }
    }
}