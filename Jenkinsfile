pipeline {
    agent any

    stages {

        stage('Fetch artifacts') {
            steps {
                cleanWs()
                script {
                    def projects = [
                        'stamp-webservices-pipeline',
                        'stamp-web-aurelia-pipeline',
                        'stamp-web-vuejs-pipeline'
                    ]

                    projects.each { project ->
                        copyArtifacts(
                            projectName: project,
                            selector: [$class: 'StatusBuildSelector', stable: false],
                            filter: 'archive/*.tgz',
                            target: 'upstream/${project}'
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
                  for pkg in "$dir/archive"/*.tgz; do
                    echo "Unpacking $pkg"
                    tar \
                      --extract \
                      --gzip \
                      --file="$pkg" \
                      --directory=stamp-web \
                      --strip-components=1 \
                      --exclude='config/application.json' \
                      --exclude='config/users.json' \
                      --exclude='config/exchange-rates.json'
                  done
                done
                echo "Moving dist resources to www static folder"
                mv -rf stamp-web/dist/* stamp-web/www
                '''
            }
        }
    }
}
