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
                        sh "npm version minor --no-git-tag-version"
                        env.NEW_VERSION = sh (script: "grep version package.json | cut -d '\"' -f4", returnStdout: true).trim()
                    }
                }
            }
        }

        stage("RunTests") {
            steps {
                script {
                    echo "----- RUNNING TESTS -----"

                    dir("app") {
                        sh "npm install"
                        sh "npm test"
                    }
                }
            }
        }

        stage("BuildImage") {
            steps {
                script {
                    echo "----- BUILDING DOCKER IMAGE -----"

                    sh "docker build -t $NEXUS_ADDRESS/nodejs-app:$NEW_VERSION ."
                }
            }
        }

        stage("PushImageToRegistry") {
            steps {
                script {
                    echo "----- PUSHING DOCKER IMAGE TO NEXUS -----"

                    withCredentials([usernamePassword(credentialsId: 'Nexus-Credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PWD')]) {
                        sh "echo $NEXUS_PWD | docker login -u $NEXUS_USER --password-stdin $NEXUS_ADDRESS"
                        sh "docker push $NEXUS_ADDRESS/nodejs-app:$NEW_VERSION"
                    }
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