def CURRENT_ENV = ""
def APP = ""

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

node {
    stage("Check Environment") {
        checkEnv()
    }
    stage("Build Apps") {
        if (CURRENT_ENV == "dev") {
          echo "BUILD API"
        } else if (CURRENT_ENV == "staging") {
          echo "BUILD STAGING"
        } else if (CURRENT_ENV == "prod") {
          echo "BUILD PROD"
        } 
    }
}
