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

                    projects.each { project ->
                        copyArtifacts(
                            projectName: project,
                            selector: [$class: 'StatusBuildSelector', stable: false],
                            filter: 'archive/*.tgz',
                            target: 'upstream'
                        )
                    }
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
                '''
            }
        }


        stage('Create Bundle') {
            steps {
                sh '''
                    VERSION=$(node -p "require('./stamp-web/package.json').version")
                    cd stamp-web
                    tar -cf ../stamp-web-${VERSION}.tgz *
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
