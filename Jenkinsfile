pipeline {
    agent any

    stages {

        stage('Fetch artifacts') {
            steps {
                cleanWs()

                copyArtifacts(
                    projectName: 'stamp-webservices-pipeline',
                    selector: lastSuccessful(),
                    filter: 'archive/*.tgz',
                    target: 'upstream/stamp-webservices-pipeline'
                )

                copyArtifacts(
                    projectName: 'stamp-web-vuejs-pipeline',
                    selector: lastSuccessful(),
                    filter: 'archive/*.tgz',
                    target: 'upstream/stamp-web-vuejs-pipeline'
                )
            }
        }

        stage('Unpack Artifacts') {
            steps {
                sh '''
                set -e

                mkdir -p app

                for dir in upstream/*; do
                  for pkg in "$dir"/*.tgz; do
                    echo "Unpacking $pkg"
                    tar \
                      --extract \
                      --gzip \
                      --file="$pkg" \
                      --directory=app \
                      --strip-components=1 \
                      --exclude='config/application.json' \
                      --exclude='config/users.json' \
                      --exclude='config/exchange-rates.json'
                  done
                done
                '''
            }
        }
    }
}
