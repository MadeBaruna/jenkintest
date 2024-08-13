import org.jenkinsci.plugins.pipeline.modeldefinition.Utils

def currentEnv = "dev"
def app = ""
def skipRemainingStages = false

def skip(name, execute, block) {
    return stage(name, !skipRemainingStages && execute ? block : {
        echo "Skipped stage $name"
        Utils.markStageSkippedForConditional(STAGE_NAME)
    })
}

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

    currentEnv = isStaging ? "staging": "prod";
    app = parts[0][1]

    echo "Current env: $currentEnv"
    echo "App: $app"
}

node {
    stage("CHECK TAG", true) {
        checkTag()
    }
    stage("DEV", currentEnv == "dev") {
        skipRemainingStages = true
        echo "BUILD DEV"
    }
    stage("STAGING", currentEnv == "staging") {
        skipRemainingStages = true
        echo "BUILD STAGING"
    }
    stage('PROD', currentEnv == "prod") {
        def userInput = input(id: 'confirm', message: 'Approve deployment to PROD?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you agree with this']])
        if (!userInput) {
            error('Deployment to PROD was not approved')
        }

        echo "BUILD PROD"
    }
}
