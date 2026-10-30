pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = 'true'
        PROJECT_NAME              = 'BlazingTrails.sln'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet restore ${PROJECT_NAME}'
                sh 'dotnet build ${PROJECT_NAME} -c Release --no-restore -v m'

                script {
                    if (env.TEST_PROJECT) {
                        sh "dotnet test ${env.TEST_PROJECT} -c Release --no-restore --logger trx"
                        junit allowEmptyResults: true, testResults: '**/*.trx'
                    }
                }
            }
        }

        stage('Publish') {
            steps {
                sh 'dotnet publish BlazingTrails.Api/BlazingTrails.Api.csproj -c Release --no-restore -o publish-output'
                archiveArtifacts artifacts: 'publish-output/**', allowEmptyArchive: true
            }
        }
    }

    post {
        // Geen cleanWs(): bin/ en obj/ blijven zodat de volgende build
        // incrementeel en daardoor een stuk sneller is.
        always {
            script {
                if (isWindows()) {
                    bat "call @echo off & rd /s /q publish-output"
                } else {
                    sh 'rm -rf publish-output'
                }
            }
        }
    }
}
