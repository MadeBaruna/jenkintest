import org.jenkinsci.plugins.pipeline.modeldefinition.Utils
import groovy.transform.Field

@Field def currentEnv = "DEV"
@Field def app = ""
@Field def skipRemainingStages = false

def stage(name, execute, block) {
    return stage(name, !skipRemainingStages && execute ? block : {
        echo "Skipped stage $name"
        Utils.markStageSkippedForConditional(STAGE_NAME)
        sleep 10
    })
}

def boolean checkTag() {
    def tag = ref.replaceAll(/refs\/.*\//, "").trim()
    
    echo "checking tag $tag"
    if (tag == "main") {
      echo "Current env: $currentEnv"
      return
    }
    def parts = tag =~ /(.*)@\d+\.\d+\.\d+(-rc)?$/
    def matches = parts.matches()
    if (!matches) {
      echo "Current env: $currentEnv"
      return
    }
    def isStaging = parts[0][2] == "-rc"

    currentEnv = isStaging ? "STAGING": "PROD";
    app = parts[0][1]

    echo "Current env: $currentEnv"
    echo "App: $app"
}

def buildApps() {
  parallel (
    api: stage("$currentEnv: API", currentEnv == "DEV" || app == "api") {
      echo "BUILD API"
    },
    queue: stage("$currentEnv: QUEUE", currentEnv == "DEV" || app == "queue") {
      echo "BUILD QUEUE"
    },
    portal: stage("$currentEnv: PORTAL", currentEnv == "DEV" || app == "portal") {
      echo "BUILD PORTAL"
    },
    landing: stage("$currentEnv: LANDING", currentEnv == "DEV" || app == "landing") {
      echo "BUILD LANDING"
    }
  )
}

node {
    stage("CHECK TAG", true) {
        checkTag()
    }
    stage("DEV", currentEnv == "DEV") {
        echo "BUILD DEV"
        buildApps()
        skipRemainingStages = true
    }
    stage("STAGING", currentEnv == "STAGING") {
        echo "BUILD STAGING"
        buildApps()
        skipRemainingStages = true
    }
    stage('PROD', currentEnv == "PROD") {
        def userInput = input(id: 'confirm', message: 'Approve deployment to PROD?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you agree with this']])
        if (!userInput) {
            error('Deployment to PROD was not approved')
        }

        echo "BUILD PROD"
        buildApps()
    }
}
