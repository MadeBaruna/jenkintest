import org.jenkinsci.plugins.pipeline.modeldefinition.Utils
import groovy.transform.Field

@Field def currentEnv = "dev"
@Field def app = ""
@Field def skipRemainingStages = false

def stage(name, execute, block) {
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

def buildApps() {
  parallel(
    api: stage("API", currentEnv == "dev" || app == "api") {
      echo "BUILD API"
    },
    queue: stage("QUEUE", currentEnv == "dev" || app == "queue") {
      echo "BUILD QUEUE"
    },
    portal: stage("PORTAL", currentEnv == "dev" || app == "portal") {
      echo "BUILD PORTAL"
    },
    landing: stage("LANDING", currentEnv == "dev" || app == "landing") {
      echo "BUILD LANDING"
    }
  )
}

node {
    stage("CHECK TAG", true) {
        checkTag()
    }
    stage("DEV", currentEnv == "dev") {
        echo "BUILD DEV"
        buildApps()
        skipRemainingStages = true
    }
    stage("STAGING", currentEnv == "staging") {
        echo "BUILD STAGING"
        buildApps()
        skipRemainingStages = true
    }
    stage('PROD', currentEnv == "prod") {
        def userInput = input(id: 'confirm', message: 'Approve deployment to PROD?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you agree with this']])
        if (!userInput) {
            error('Deployment to PROD was not approved')
        }

        echo "BUILD PROD"
        buildApps()
    }
}
