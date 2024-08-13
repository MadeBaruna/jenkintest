def CURRENT_ENV = "dev"
def APP = ""
def SKIP_REMAINING_STAGES = false

def boolean checkTag() {
    def tag = ref.replaceAll(/ref\/.*\//, "").trim()
    
    echo "checking tag $tag"
    if (tag == "main") {
      echo "Current env: $CURRENT_ENV"
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

node {
    stage("CHECK TAG") {
        checkTag()
    }
    stage("DEV") {
        if (CURRENT_ENV != "dev") {
          return
        }
        SKIP_REMAINING_STAGES = true
        echo "BUILD DEV"
    }
    stage("STAGING") {
        if (CURRENT_ENV != "staging") {
          return
        }
        SKIP_REMAINING_STAGES = true
        echo "BUILD STAGING"
    }
    stage('PROD') {
        if (CURRENT_ENV != "prod") {
          return
        }
        def userInput = input(id: 'confirm', message: 'Approve deployment to PROD?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you agree with this']])
        if (!userInput) {
            error('Deployment to PROD was not approved')
        }

        echo "BUILD PROD"
    }
}
