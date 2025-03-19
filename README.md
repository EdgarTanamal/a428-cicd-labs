# a428-cicd-labs
Repository untuk Kelas Belajar Implementasi CI/CD
Step by step

 1. Running Jenkins in Docker

2. Run Docker-in-Docker (DinD) container:
docker run --name jenkins-docker --detach --privileged --network jenkins --network-alias docker --env DOCKER_TLS_CERTDIR=/certs --volume jenkins-docker-certs:/certs/client --volume jenkins-data:/var/jenkins_home --publish 2376:2376 --publish 3000:3000 --restart always docker:dind --storage-driver overlay2

3. Create dockerfile:
FROM jenkins/jenkins:2.346.1-jdk11
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \  
    https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
    signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
    https://download.docker.com/linux/debian \
    $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.5 docker-workflow:1.28"

4.Build the image
docker build -t myjenkins-blueocean:2.426.1 .

5.Run Jenkins container:
docker run --name jenkins-blueocean --detach --network jenkins --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --publish 8080:8080 --publish 50000:50000 --volume jenkins-data:/var/jenkins_home --volume jenkins-docker-certs:/certs/client:ro --volume "C:/Users/edgar:/home" --volume "D:/a428-cicd-labs:/home/a428-cicd-labs" --restart=on-failure --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" myjenkins-blueocean:2.426.3

6.Accessing Jenkins
Open `http://localhost:8080` in a browser.

7. Check password 
docker logs jenkins-blueocean

8. Fork and Clone React App Repository
git clone -b react-app https://github.com/YOUR-GITHUB-USERNAME/a428-cicd-labs.git

9.Create a Jenkins Pipeline
Create `Jenkinsfile` in the project root with the following content:pipeline {
    agent {
        docker {
            image 'node:16-buster-slim'
            args '-p 3000:3000'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
    }
}

10.Run the Pipeline
Open Jenkins > Blue Ocean > Run Pipeline > Check the logs for execution details.

