# Solutions for Exercises-07 Container-with-Docker

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

COPY . .

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
