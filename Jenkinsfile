def runCondition(project) {
    return sh(returnStatus: true, label: "Checking if $project changed since last time", script: """
      set +x
      git diff --name-only HEAD~1 | grep --quiet '^$project/.*'
      set -x
    """) == 0 || params.RUN_ALL
}

pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_ALL', defaultValue: false, description: 'Force run all stages')
    }

    environment {
        SONARQUBE_URL = 'sonarqube:9000'
        SONAR_TOKEN = 'sqp_0f5cf99a252de45ed5cc53e645d53dd0ce08d47c'
        PROJECT_KEY = 'digger'
    }

    stages {
        stage("Initialize Shared Libraries") {
            steps {
                library identifier: 'custom@master', retriever: modernSCM(
                    [$class: 'GitSCMSource',
                    remote: 'git@repo:/git-server/repos/shared-lib',
                    credentialsId: 'local-repo']
                )
            }
        }
        stage('Build project') {
            steps {
                script {
                    deployFunc version: '3.10.3', runtime: 'python'
                }
            }
        }

        stage("SAST") {
            sh """
                docker run \
                    --rm \
                    -e SONAR_HOST_URL='http://${SONARQUBE_URL}' \
                    -e SONAR_SCANNER_OPTS='-Dsonar.projectKey=${PROJECT_KEY}' \
                    -e SONAR_TOKEN='sqp_0f5cf99a252de45ed5cc53e645d53dd0ce08d47c' \
                    -v '$(pwd)/:/usr/src' \
                    sonarsource/sonar-scanner-cli
            """
        }
    }
}