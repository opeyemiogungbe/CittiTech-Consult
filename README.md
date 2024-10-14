# CI-CD-Deployment-of-ecommerce-website-with-jenkins

## Overview
This project demonstrates a complete CI/CD pipeline using Jenkins to automate web application deployment. Jenkins is integrated with GitHub as the source code management tool, while Docker containerizes the application. The final Docker image is pushed to Docker Hub for future deployments. The pipeline ensures continuous integration, and continuous delivery, and automates deployment to ensure scalability and reliability.

### Table of Contents

**Prerequisites**

**Architecture**

- Step 1: Install Jenkins
- Step 2: Set up Necessary Jenkins Plugins
- Step 3: Configure Jenkins Security
- Step 4: Integrate Jenkins with GitHub
- Step 5: Configure Webhooks for Jenkins Builds
- Step 6: Create Jenkins Freestyle Job
- Step 7: Create Jenkins Pipeline Script
- Step 8: Build Docker Images in Jenkins
- Step 9: Running the Docker Container
- Step 10: Push Docker Images to a Registry
- Step 11: Documentation

**Prerequisites**

Before we begin, we must ensure we have the following:

- Ubuntu 20.04 LTS (or similar Linux distro)
- Jenkins (latest stable version)
- GitHub account with repository for the project
- Docker (installed on the Jenkins server)
- Docker Hub account for image storage
- Jenkins Plugins:
- Git Plugin
- Docker Pipeline Plugin
- Pipeline Plugin
- GitHub Plugin

### Architecture

![image](https://github.com/user-attachments/assets/e3cacde8-d12c-4df6-ba44-5c419e38a8cf)

This pipeline works as follows:

we are going to push our code changes to the GitHub repository.
- A GitHub webhook will trigger a Jenkins build.
- Jenkins checks out the code and runs unit tests.
- Jenkins builds a Docker image of the application.
- Jenkins pushes the Docker image to Docker Hub.
- The new image can be deployed on any server using Docker.

## Step 1: Install Jenkins
To begin, we need to install Jenkins on a dedicated server. Jenkins will act as the orchestrator for your CI/CD pipeline.

1. Install Java, as Jenkins requires Java to run:
```
sudo apt update
sudo apt install openjdk-11-jdk -y
```
2. Add Jenkins Repository and install Jenkins:
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
```
3. Start Jenkins and enable it to start at boot:
```
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

4. Access Jenkins:
Open port 8080 in inbound rules on jenkins server to enable us access it on web browser

![Screenshot 2023-11-08 083515](https://github.com/user-attachments/assets/b7b44a56-bd28-4e49-8e03-4d45b707a2b3)

Open Jenkins in a web browser: http://<your-server-ip>:8080

We are going to cat into /var/lib/jenkins/secrets/initialAdminPassword to get our password

## Step 2: Set up Necessary Jenkins Plugins

Once Jenkins is installed, install the following plugins:

- Git Plugin (to integrate with GitHub).
- Docker Pipeline Plugin (to enable Docker commands within Jenkins).
- Pipeline Plugin (to write pipeline-as-code with Jenkinsfile).
- GitHub Plugin (to integrate webhooks from GitHub).

Navigate to:

Manage Jenkins > Manage Plugins > Available and search for the required plugins.

## Step 3: Configure Jenkins Security

1. Create a Jenkins User to avoid using the default admin account:

  - Go to Manage Jenkins > Manage Users > Create User.

2. Set Global Security Settings:

  - Manage Jenkins > Configure Global Security
  - Enable Jenkins' own user database for security.
  - Configure roles and permissions as needed.

3. Add Credentials:

- You’ll need to store credentials for GitHub, Docker Hub, and possibly other services like cloud providers.

  Go to Manage Jenkins > Manage Credentials to add credentials such as GitHub access tokens and Docker Hub login details.
  
## Step 4: Integrate Jenkins with GitHub

Jenkins needs to pull your source code from GitHub.

1. Install GitHub Plugin via the Jenkins plugin manager.

2. Create a GitHub Repository and clone it:
  ```
  git clone https://github.com/yourusername/yourapp.git
  ```
3. Configure GitHub in Jenkins Job:
  - When creating a Jenkins job, under "Source Code Management", select Git.
  - Provide the GitHub repository URL.
  - Add credentials (Personal Access Token) for authentication.

## Step 5: Configure Webhooks for Jenkins Builds

To automate the build trigger:

1. Go to Your GitHub Repository:

  - Navigate to Settings > Webhooks > Add webhook.
  - Set the Payload URL to http://your-jenkins-url:8080/github-webhook/.

  - In Jenkins, configure the job to trigger builds using webhooks:
    Check the box for GitHub hook trigger for GITScm polling.

## Step 6: Create Jenkins Freestyle Job
A Freestyle Job can be used to build the web application and run unit tests:

1. Go to Jenkins Dashboard and click New Item > Freestyle Project.

2. Set the GitHub Repository under "Source Code Management".

3. Add Build Steps to run unit tests or any other tasks

4. Configure Post-build Actions to archive artifacts or notify teams.

## Step 7: Create Jenkins Pipeline Script

This is a more advanced setup where you define a Jenkins pipeline using code.

1. Create a New Pipeline Job:

  - Go to New Item > Pipeline.

2.Write the Pipeline Script:

```
pipeline {
    agent any

    stages {
        stage('Connect To Github') {
            steps {
                checkout([$class: 'GitSCM', 
                  branches: [[name: '*/main']],
                  doGenerateSubmoduleConfigurations: false,
                  extensions: [],
                  userRemoteConfigs: [[url: 'https://github.com/opeyemiogungbe/CittiTech-Consult.git']]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t dockerfile .'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -itd -p 8081:80 dockerfile'
                }
            }
        }
    }
}
```
![Screenshot 2024-07-14 075144](https://github.com/user-attachments/assets/5b2b32b7-be23-4d8e-b004-22ce10c48180)

![Screenshot 2024-07-14 075202](https://github.com/user-attachments/assets/be60efed-3645-4d89-a559-d1c142387c9b)

![Screenshot 2024-07-14 075217](https://github.com/user-attachments/assets/b380b638-33ac-4dee-a8aa-8b04e6744a80)

3. We are going to commit the Jenkinsfile to the root of our GitHub repository.

# Step 8: Build Docker Images in Jenkins

In the pipeline, we’ll need to build Docker images.

1. Install Docker on our Jenkins server:
   - We are going to create docker.sh files at the root folder and open it up with vi/vim
   - we are going put in all the necesarry docker dependencies and download docker

![Screenshot 2024-07-14 045118](https://github.com/user-attachments/assets/37b2d4d0-8864-43d1-a389-54e817a544a5)

```
# Docker Installation
sudo apt update -y
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu/ $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl status docker
```
![Screenshot 2024-07-14 045229](https://github.com/user-attachments/assets/c74e79e6-16ae-400e-954b-111531114eef)

- We are going to set permisson for the docker file so it can be executable and run the file
```
chmod u+x docker.sh (for permission)
./docker.sh
```

![Screenshot 2024-07-07 103547](https://github.com/user-attachments/assets/67ec8acc-e3d8-42f2-903b-ce72f20c01ef)

- We are going to restart jenking and docker to make sure everything works our installation was successful

![Screenshot 2024-07-14 062401](https://github.com/user-attachments/assets/7ec0a15c-212a-497e-be86-9af78b58f141)

![Screenshot 2024-07-07 103621](https://github.com/user-attachments/assets/bae4ff25-720d-42b2-b9e4-7b12975007b8)

Now we can see our installation is successful 

2. Add a Docker Build Step:
    ```
    docker build -t your-app:latest
    ```
    this build our container

## Step 9: Running the Docker Container
To deploy the application:

1. Run the Container:
  ```
  docker run -d -p 80:80 your-app:latest
  ```
![Screenshot 2024-07-15 141020](https://github.com/user-attachments/assets/00ff28bb-8a5a-40b8-859d-626d68f22093)

We can see our docker is live and running

2. Access the Application:

  - Open your browser and visit http://<your-server-ip> to view the running application.

![Screenshot 2024-07-14 084610](https://github.com/user-attachments/assets/30798ffa-d042-42c3-816f-aa85bc8366aa)

A simple html and style.css was addecommerce website and we can see its live and working .


## Step 10: Push Docker Images to a Registry

1. Add Docker Hub Credentials to Jenkins:

  - Manage Jenkins > Manage Credentials > Global > Add Credentials.

2. Push the Docker Image to Docker Hub within the pipeline:

```
docker login -u your-dockerhub-username -p your-password
docker tag your-app:latest your-dockerhub-username/your-app:latest
docker push your-dockerhub-username/your-app:latest
```

- Docker Image Creation:
The Jenkins pipeline handles Docker image creation with 
```
docker build -t your-app:latest ..
```

- Running a Container:
After building, the container can be started using
```
docker run -d -p 80:80 your-app:latest.
```
- Pushing Images to Docker Hub:
The final Docker image is pushed using:
```
docker push your-dockerhub-username/your-app:latest
```

By following this guide, you will have a fully functional CI/CD pipeline set up for automating your web application deployment using Jenkins and Docker.
