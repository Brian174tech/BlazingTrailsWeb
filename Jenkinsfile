pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOTNET_SKIP_FIRST_TIME_EXPERIENCE = 'true'
        DOTNET_CLI_TELEMETRY_OPTOUT      = 'true'
        PROJECT_NAME                     = 'BlazingTrails.sln'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore ${PROJECT_NAME}'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build ${PROJECT_NAME} -c Release --no-restore'
            }
        }

        stage('Test') {
            when {
                expression { return env.TEST_PROJECT != null }
            }
            steps {
                sh "dotnet test ${TEST_PROJECT} -c Release --no-restore --logger trx"
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/*.trx'
                }
            }
        }

        stage('Publish') {
            steps {
                sh 'dotnet publish BlazingTrails.Api/BlazingTrails.Api.csproj -c Release --no-restore -o publish-output'
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'publish-output/**', allowEmptyArchive: true
            }
        }
    }

    post {
        success {
            echo 'Build geslaagd!'
        }
        failure {
            echo 'Build is mislukt.'
        }
        always {
            cleanWs()
        }
    }
}
