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
