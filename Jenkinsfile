// Configure the Pipeline from SCM job to track */main (see docs/ci.md).
def dotnetCommand(String arguments) {
    if (isUnix()) {
        sh "dotnet ${arguments}"
    } else {
        bat "dotnet ${arguments}"
    }
}

pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timeout(time: 15, unit: 'MINUTES')
    }

    triggers {
        githubPush()
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Build and test main') {
            when {
                expression {
                    env.BRANCH_NAME ? env.BRANCH_NAME == 'main' :
                        env.GIT_BRANCH in ['main', 'origin/main', 'refs/heads/main']
                }
            }
            stages {
                stage('Restore') {
                    steps {
                        script {
                            dotnetCommand('restore Homies.sln')
                        }
                    }
                }

                stage('Build') {
                    steps {
                        script {
                            dotnetCommand('build Homies.sln --configuration Release --no-restore')
                        }
                    }
                }

                stage('Test') {
                    steps {
                        dir('TestResults') {
                            deleteDir()
                        }
                        script {
                            dotnetCommand('test Homies.sln --configuration Release --no-build --no-restore --logger trx --results-directory TestResults')
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'TestResults/**/*.trx', allowEmptyArchive: true
                        }
                    }
                }
            }
        }
    }
}
