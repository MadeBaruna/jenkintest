import groovy.transform.Field

@Field def skipRemainingStages = false
@Field def CURRENT_ENV = ""
@Field def APP = ""

def boolean checkEnv() {
    def tag = ref.replaceAll(/ref\/.*\//, "").trim()
    
    echo "checking tag $tag"
    if (tag == "main") {
        return "dev"
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

def buildApps() {
    return {
        stage("API") {
            when {
                expression {
                    return CURRENT_ENV == "dev" || APP == "api"
                }
            }
            steps {
                echo "BUILD API"
            }
        }
        stage("QUEUE") {
            when {
                expression {
                    return CURRENT_ENV == "dev" || APP == "queue"
                }
            }
            steps {
                echo "BUILD QUEUE"
            }
        }
        stage("PORTAL") {
            when {
                expression {
                    return CURRENT_ENV == "dev" || APP == "portal"
                }
            }
            steps {
                echo "BUILD PORTAL"
            }
        }
        stage("LANDING") {
            when {
                expression {
                    return CURRENT_ENV == "dev" || APP == "landing"
                }
            }
            steps {
                echo "BUILD LANDING"
            }
        }
        stage("SKIP") {
            when {
                expression {
                    return CURRENT_ENV != "prod"
                }
            }
            steps {
                skipRemainingStages = true
            }
        }
    }
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
            stages buildApps
        }
        stage('STAGING') {
            when {
                expression {
                    return !skipRemainingStages && CURRENT_ENV == "staging"
                }
            }
            stages buildApps
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
            stages buildApps
        }
    }
}