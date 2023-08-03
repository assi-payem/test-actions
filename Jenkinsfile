def parallelStagesMap

def generateStage(job) {
    return {
        stage("build: ${job}") {
            current_stage = env.STAGE_NAME
            script {
                sh """cd ${job}
                docker build . -t ${job}
                """
            }
        }
    }
}

pipeline {
    agent {
        label 'master'
    }
    options {
        ansiColor('xterm')
        timeout(time: 30, unit: 'MINUTES')
    }
    environment {
        SERVICES = ""
    }
    stages {
        stage('Set Parallel') {
            steps {
                script {
                    current_stage = env.STAGE_NAME
                    def changedFolders = sh(script: 'git diff --name-only ${GIT_COMMIT} ${GIT_PREVIOUS_COMMIT} | awk -F / \'NF{NF--};1\' | sort -u', returnStdout: true).trim()
                    echo "Changed folders: ${changedFolders}"
                    SERVICES = gitChangedDirs().split(',')
                    println SERVICES
                    parallelStagesMap = SERVICES.collectEntries {
                        ["${it}" : generateStage(it)]
                    }
                }
            }
        }
        stage('Parallel build') {
            steps {
                script {
                    parallel parallelStagesMap
                }
            }
        }
    }
    post {
        success {
            script {
                println "All good"
            }
        }
        failure {
            script {
                println "Didn't work"
            }
        }
        aborted {
            script {
                println "Aborted"
            }
        }
    }
}
