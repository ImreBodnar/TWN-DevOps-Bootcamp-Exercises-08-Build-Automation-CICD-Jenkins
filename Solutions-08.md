# Solutions for Exercises-08-Build-Automation-with-Jenkins

## Exercise 0

```bash
# Navigate to the working directory.
cd Repositories

# Cloning.
git clone https://gitlab.com/twn-devops-bootcamp/latest/08-jenkins/jenkins-exercises.git

# Deleting .git folder.
rm -rf jenkins-exercises/.git

# Renaming the folder, because I want to save it to my GitHub repository with a different name.
mv jenkins-exercises TWN-DevOps-Bootcamp-Exercises-08-Build-Automation-CICD-Jenkins
cd TWN-DevOps-Bootcamp-Exercises-08-Build-Automation-CICD-Jenkins

git init
git status

# Commit & push.
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/ImreBodnar/TWN-DevOps-Bootcamp-Exercises-08-Build-Automation-CICD-Jenkins.git
git push -u origin main
```

## Exercise 1

**The newly created Dockerfile:**

```bash
FROM node:24-alpine

WORKDIR /usr/app

COPY app/package*.json ./

RUN npm install --production

COPY app/ .

EXPOSE 3000

CMD ["node", "server.js"]
```

**I used this command to build the image:**

```bash
docker build -t nodejs-app:1.0 .
```

## Exercise 2

**This is the content of the Jenkinsfile I created:**

```groovy
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
                        sh 'npm version minor --no-git-tag-version'
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
                        sh 'npm install'
                        sh 'npm test'
                    }
                }
            }
        }

        stage("BuildImage") {
            steps {
                script {
                    echo "----- BUILDING DOCKER IMAGE -----"

                    sh 'docker build -t $NEXUS_ADDRESS/nodejs-app:$NEW_VERSION .'
                }
            }
        }

        stage("PushImageToRegistry") {
            steps {
                script {
                    echo "----- PUSHING DOCKER IMAGE TO NEXUS -----"

                    withCredentials([usernamePassword(credentialsId: 'Nexus-Credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PWD')]) {
                        sh 'echo $NEXUS_PWD | docker login -u $NEXUS_USER --password-stdin $NEXUS_ADDRESS'
                        sh 'docker push $NEXUS_ADDRESS/nodejs-app:$NEW_VERSION'
                    }
                }
            }
        }

        stage("CommitNewVersion") {
            steps {
                script {
                    echo "----- COMMITING & PUSHING NEW VERSION TO GIT -----"

                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PWD')]) {
                        sh 'git config user.email jenkins.ci@example.com'
                        sh 'git config user.name JenkinsCI'
                        sh 'git add .'
                        sh 'git commit -m "CI pipeline: Version Increment"'
                        sh 'git push https://$GIT_USER:$GIT_PWD@github.com/$GIT_USER/TWN-DevOps-Bootcamp-Exercises-08-Build-Automation-CICD-Jenkins.git HEAD:main'
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
```

**The Docker Image is built and uploaded into Nexus by my CI/CD pipeline:**

![The docker image is built and uploaded into nexus by cicd pipeline.](Exercise-2-Docker-Image-Uploaded-to-Nexus.jpg)

## Exercise 3

> To run the newly built Docker Image by my pipeline, I used my Jenkins server droplet.

**I used these commands to run the newly build Docker Image by my pipeline:**

```bash
ssh -i ~/.ssh/digital_ocean_key root@JENKINS_ADDRESS

docker login NEXUS_ADDRESS:8082
docker run -d -p 8085:3000 --name MyNodeJsApp NEXUS_ADDRESS:8082/nodejs-app:1.3.0
docker ps
```

## Exercise 4

**I add a new GitHub repository for my Jenkins shared library:**

```bash
cd Repositories
mkdir TWN-DevOps-Bootcamp-Exercises-08-Build-Automation-CICD-Jenkins-Groovy
cd TWN-DevOps-Bootcamp-Exercises-08-Build-Automation-CICD-Jenkins-Groovy

# I moved the code here from the Jenkinfile...

git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/ImreBodnar/TWN-DevOps-Bootcamp-Exercises-08-Build-Automation-CICD-Jenkins-Groovy.git
git push -u origin main
```

**The new shared Jenkins library:**

![The new shared jenkins library.](Exercise-4-1-The-New-Shared-Jenkins-Library.jpg)

**You can find this GitHub repository here:**
<https://github.com/ImreBodnar/TWN-DevOps-Bootcamp-Exercises-08-Build-Automation-CICD-Jenkins-Groovy>

**This is the updated Jenkinsfile:**

```groovy
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
```

**I added the new shared library to the Jenkins settings:**

![I added the new shared library to the jenkins settings](Exercise-4-2-The-Shared-Library-in-Jenkins-Settings.jpg)

**Some successful runs in Jenkins:**

![Some successful runs in jenkins.](Exercise-4-3-Some-Successful-Runs-in-Jenkins.jpg)
