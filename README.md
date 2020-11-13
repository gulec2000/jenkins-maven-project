# jenkins-maven-project
```text
Building Jenkins Pipelines on Amazon Linux 2 AWS EC2 Instance
```
```text 
        Configuring Jenkins Pipeline with GitHub Webhook to Build the a Java Maven Project
```

- To build the `java` maven project with Jenkins pipeline using the `Jenkinsfile` and `GitHub Webhook`. To accomplish this task, we need;

  - a java code to build

  - a java environment to run the build stages on the java code

  - a maven environment to run the build stages on the java code

  - a Jenkinsfile configured for an automated build on our repo

- Create a public project repository `jenkins-maven-project` on your GitHub account.

- Clone the `jenkins-maven-project` repository on local computer.

- Copy the files from my repository(https://github.com/gulec2000/jenkins-maven-project/tree/main/hello-app)  [hello-app](./hello-app)  and paste under the `jenkins-maven-project` your  GitHub repo folder.

- Go to the Jenkins dashboard and click on `New Item` to create a pipeline.

- Enter `pipeline-with-jenkinsfile-and-webhook-for-maven-project` then select `Pipeline` and click `OK`.

- Enter `Simple pipeline configured with Jenkinsfile and GitHub Webhook for Maven project` in the description field.

- Put a checkmark on `GitHub Project` under `General` section, enter URL of the project repository.

```text
https://github.com/gulec2000/jenkins-maven-project.git
```

- Put a checkmark on `GitHub hook trigger for GITScm polling` under `Build Triggers` section.

- Go to the `jenkins-maven-project` repository page and click on `Settings`.

- Click on the `Webhooks` on the left hand menu, and then click on `Add webhook`.

- Copy the Jenkins URL from the AWS Management Console, paste it into `Payload URL` field, add `/github-webhook/` at the end of URL, and click on `Add webhook`.

```text
http://ec2-54-144-151-76.compute-1.amazonaws.com:8080/github-webhook/

- Go to the `Pipeline` section, and select `Pipeline script from SCM` in the `Definition` field.

- Select `Git` in the `SCM` field.

- Enter URL of the project repository, and let others be default.

```text
https://github.com/gulec2000/jenkins-maven-project.git
```

- Click `apply` and `save`. Note that the script `Jenkinsfile` should be placed under root folder of repo.

# Java Installation
```bash
- sudo yum update -y
- sudo amazon-linux-extras install java-openjdk11 -y
- sudo yum install java-devel 
```
# Maven Settings

- 'Maven' Installation on 'ec2 Instance' and setting uo the environment
```bash
- cd /opt
- wget https://www-eu.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
- sudo tar xzf apache-maven-3.6.3-bin.tar.gz
- sudo ln -s apache-maven-3.6.3 maven
- sudo echo 'export M2_HOME=/opt/maven' >> /etc/profile.d/maven.sh
- sudo echo 'export PATH=${M2_HOME}/bin:${PATH}' >> /etc/profile.d/maven.sh
- source /etc/profile.d/maven.sh
```
# GIT installation on 'ec2 instance'
```bash
 - sudo yum install git -y
```
# Open Jenkins GUI on web browser
- Setting System Maven Path for default usage
```text
- Go to `Manage Jenkins`
  - Select `Configure System`
  - Find `Environment variables` part,
  - Click `Add`
    - for `Name`, enter `PATH+EXTRA` 
    - for `Value`, enter `/opt/maven/bin`
```
- Setting a 'specific Maven Release' in Jenkins for usage
```text
- Go to the `Global Tool Configuration`

- To the bottom, `Maven` section
  - Give a name such as `maven-3.6.3`
  - Select `install automatically`
  - `Install from Apache` version `3.6.3`
```
- Since the Jenkins Server would be  running on Docker Machine, we could leverage from docker image of maven to setup the java and maven environment.

- Create a `Jenkinsfile` with the following pipeline script. This is for 'Docker image'

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3-openjdk-8'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') {
            steps {
                sh 'chmod +x deliver-script.sh'
                sh './deliver-script.sh'
            }
        }
    }
}
```

- For 'native structured Jenkins Server'

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn -f hello-app/pom.xml -B -DskipTests clean package'
            }
            post {
                success {
                    echo "Now Archiving the Artifacts....."
                    archiveArtifacts artifacts: '**/*.jar'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -f hello-app/pom.xml test'
            }
            post {
                always {
                    junit 'hello-app/target/surefire-reports/*.xml'
                }
            }
        }
    }
}
```

- Commit and push the changes to the remote repo on GitHub.

```bash
git add .
git commit -m 'added jenkinsfile and maven project'
git push
```

- Observe the new built triggered with `git push` command on the Jenkins project page.

- Explain the role of the docker image of maven, `Jenkinsfile` and GitHub Webhook in this automation.
