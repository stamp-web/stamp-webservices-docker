pipeline {
    agent any

    triggers {
        upstream(upstreamProjects: 'stamp-web-vuejs,stamp-web-aurelia,stamp-webservices', threshold: hudson.model.Result.SUCCESS)
        githubPush()
    }

    options {
         disableConcurrentBuilds()
         buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Fetch Artifacts') {
            steps {
                cleanWs()
                script {
                    def projects = [
                        'stamp-webservices',
                        'stamp-web-aurelia',
                        'stamp-web-vuejs'
                    ]

                    def buildNumbers = [:]

                    projects.each { project ->
                        copyArtifacts(
                            projectName: project,
                            selector: [$class: 'StatusBuildSelector', stable: false],
                            filter: 'archive/*.tgz',
                            target: 'upstream'
                        )
                        buildNumbers[project] = getProjectBuildNumber(project)
                    }

                    def jsonLines = []
                    buildNumbers.each { k, v ->
                        jsonLines.add("  \"${k}\": ${v}")
                    }
                    def jsonText = "{\n" + jsonLines.join(",\n") + "\n}"
                    writeFile file: 'build-number.json', text: jsonText
                }
            }
        }

        stage('Unpack Artifacts') {
            steps {
                sh '''
                set -e
                echo "Creating stamp-web base folder"
                mkdir -p stamp-web

                for dir in upstream/*; do
                  for pkg in "$dir"/*.tgz; do
                    filename=$(basename "$pkg" .tgz)
                    package_name=$(echo "$filename" | sed 's/-[0-9].*//')
                    mkdir -p "stamp-web/$package_name"
                    tar \
                      --extract \
                      --gzip \
                      --file="$pkg" \
                      --directory=stamp-web/$package_name \
                      --strip-components=1 \
                      --exclude='config/application.json' \
                      --exclude='config/users.json' \
                      --exclude='config/exchange-rates.json'
                  done
                done
                '''
            }
        }

        stage('Process Artifacts') {
            steps {
                sh '''
                set -e
                echo $PWD
                echo "Moving dist resources to www static folder"
                mkdir -p stamp-web/www/aurelia
                mkdir -p stamp-web/www/stamp-web

                cp -rf stamp-web/stamp-webservices/* stamp-web
                cp -rf stamp-web/stamp-web-aurelia/dist/* stamp-web/www/aurelia
                cp -rf stamp-web/stamp-web-vuejs/dist/* stamp-web/www/stamp-web

                rm -rf stamp-web/stamp-webservices
                rm -rf stamp-web/stamp-web-vuejs
                rm -rf stamp-web/stamp-web-aurelia

                echo "Staging build-number.json into www folder"
                cp -f build-number.json stamp-web/www/build-number.json
                rm -f build-number.json
                '''
            }
        }


        stage('Create Bundle') {
            steps {
                sh '''
                    VERSION=$(node -p "require('./stamp-web/package.json').version")
                    cd stamp-web
                    tar -cf ../stamp-web-${VERSION}.tgz *
                    tar -cf ../stamp-web.tgz *
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'stamp-web*.tgz', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Copy archived artifacts to external location
                    copyArtifacts(
                        projectName: env.JOB_NAME,
                        selector: specific(env.BUILD_NUMBER.toString()),
                        target: '${DEPLOY_PATH}/stage/'
                    )
                }
            }
        }

    }

}

def getProjectBuildNumber(projectName) {
    def buildNumber = 0

    // 1. Try Jenkins Java API (requires script approval, but native)
    try {
        def job = jenkins.model.Jenkins.get().getItemByFullName(projectName)
        if (job) {
            def lastBuild = job.getLastBuild()
            if (lastBuild) {
                buildNumber = lastBuild.getNumber()
            }
        }
    } catch (Throwable t) {
        // sandbox or permission issue
    }

    // 2. Try REST API via curl (works if Jenkins URL is accessible from agent)
    if (buildNumber == 0) {
        try {
            def jenkinsUrl = env.JENKINS_URL
            if (jenkinsUrl) {
                def text = sh(
                    script: "curl -s -f ${jenkinsUrl}job/${projectName}/lastBuild/buildNumber",
                    returnStdout: true
                ).trim()
                if (text && text.isInteger()) {
                    buildNumber = text.toInteger()
                }
            }
        } catch (Throwable t) {
            // failed or unauthorized
        }
    }

    // 3. Fallback: Parse from filename if possible
    if (buildNumber == 0) {
        try {
            def filename = sh(
                script: "find upstream -name '${projectName}*.tgz' -exec basename {} .tgz \\; | head -n 1",
                returnStdout: true
            ).trim()
            if (filename) {
                def hyphenIndex = filename.lastIndexOf('-')
                if (hyphenIndex != -1) {
                    def buildNumPart = filename.substring(hyphenIndex + 1)
                    if (buildNumPart.isInteger()) {
                        buildNumber = buildNumPart.toInteger()
                    }
                }
            }
        } catch (Throwable t) {
            // ignore
        }
    }

    // 4. Default to a dummy placeholder so it doesn't break the build
    if (buildNumber == 0) {
        buildNumber = 1
    }

    return buildNumber
}
