#!/user/bin/env groovy

@Library('TWN-Jenkins-Shared-Library')_

pipeline {
    agent any

    tools {
        nodejs "my-nodejs"
    }

    environment {
        NEXUS_ADDRESS = "161.35.197.245:8082"
    }

    stages {
        stage("IncrementVersion") {
            steps {
                script {
                    echo "----- INCREMENTING VERSION -----"

                    dir("app") {
                        IncrementVersion()
                    }
                }
            }
        }

        stage("RunTests") {
            steps {
                script {
                    echo "----- RUNNING TESTS -----"

                    dir("app") {
                        RunNpmTests()
                    }
                }
            }
        }

        stage("BuildImage") {
            steps {
                script {
                    echo "----- BUILDING DOCKER IMAGE -----"

                    BuildDockerImage()
                }
            }
        }

        stage("PushImageToRegistry") {
            steps {
                script {
                    echo "----- PUSHING DOCKER IMAGE TO NEXUS -----"

                    PushDockerImageToRegistry()
                }
            }
        }

        stage("CommitNewVersion") {
            steps {
                script {
                    echo "----- COMMITING & PUSHING NEW VERSION TO GIT -----"

                    CommitNewVersion()
                }
            }
        }
    }

    post {
        always {
            echo "END OF PIPELINE EXECUTION."
        }

        success {
            echo "THE PIPELINE FINISHED SUCCESSFULLY!"
        }

        failure {
            echo "SOME ERRORS OCCURED!"
        }
    }
}