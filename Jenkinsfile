def parallelStagesMap

def generateStage(job) {
    return {
        stage("build: ${job}") {
            current_stage = env.STAGE_NAME
            script {
                sh """cd ${job}
                docker build . -t ${job} || echo "No Dockerfile found"
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
                    // def changedFolders = sh(script: 'git diff --name-only ${GIT_COMMIT} ${GIT_PREVIOUS_COMMIT} | awk -F / \'NF{NF--};1\' | sort -u', returnStdout: true).trim()
                    // echo "Changed folders: ${changedFolders}"
                    // .split(',')
                    // GIT_CHANGES = gitChangedDirs()
                    def baseServices = "infra,scripts"
                    GIT_CHANGES = sh(script: 'git diff --name-only ${GIT_COMMIT} ${GIT_PREVIOUS_COMMIT} | awk -F / \'NF{NF--};1\' | sort -u', returnStdout: true).trim()
                    if (GIT_CHANGES.toString() == "") {
                        building = false
                        println("myVar is empty")
                    } else {
                        building = true
                        SERVICES = GIT_CHANGES.split(',')
                        for (item in SERVICES) {
                            println item
                            if (baseServices.split(',').contains(item)) {
                                println "found"
                            } else {
                                println "not"
                            }
                        }
                        println("myVar is not empty")
                        println GIT_CHANGES.split(',')
                        SERVICES = GIT_CHANGES.split(',')
                        parallelStagesMap = SERVICES.collectEntries {
                            ["${it}" : generateStage(it)]
                        }
                    }
                }
            }
        }
        stage('Parallel build') {
            when { expression { building == true } }
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
