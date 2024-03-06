![Alt Text](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExam94enRlemhiNXk4ZWJza2Zxa2lpOTdnODVxcWk2aXdyMm0za3pvaiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/EclFLfeJvr6IPISFXJ/giphy.gif)


# Deploying Netflix Clone on EKS Cluster with Jenkins CICD and Monitoring Tools 

![Alt Text](./assets/Screenshot%202024-03-05%20at%207.44.37%20PM.png)


# Project Summary

In this project, we'll be diving into deploying a Netflix clone. I'm excited to use Jenkins as our go-to CI/CD tool, ensuring smooth deployment onto Docker containers. Additionally, we'll keep an eye on Jenkins' performance with Grafana, Prometheus, and Node Exporter. I'm hoping this detailed blog will be a valuable resource for all!


#### Phase 1: Initial Setup and Deployment

## Step 1: Launch EC2 (Ubuntu 22.04)

- Provision an EC2 instance on AWS with Ubuntu 22.04.

- Connect to the instance using SSH.



![Alt Text](./assets/Screenshot%202024-03-05%20at%2010.04.55%20PM.png)
![Alt Text](./assets/Screenshot%202024-03-05%20at%2010.06.00%20PM.png)

## Step 2: Open the Following Ports from the Inbound Rules

- 8080: Jenkins
- 9090: Prometheus
- 9100: Prometheus Metrics
- 3000: Grafana
- 9000: SonarQube
- 8081: Application

![Alt Text](./assets/Screenshot%202024-03-06%20at%2012.50.51%20AM.png)



## Step 3:
Clone the Code:

- Update all the packages and then clone the code.

-  ```sudo apt update -y```

- Clone your application's code repository onto the EC2 instance:

- ```git clone https://github.com/N4si/DevSecOps-Project.git```

![Alt Text](./assets/Screenshot%202024-03-05%20at%2010.35.50%20PM.png)


## Step 4:

- Install Docker and Run the App Using a Container:

- Set up Docker on the EC2 instance:

- ```sudo apt-get update```


![Alt Text](./assets/Screenshot%202024-03-05%20at%2010.47.29%20PM.png)


- ```sudo apt-get install docker.io -y```


![Alt Text](./assets/Screenshot%202024-03-05%20at%2010.50.00%20PM.png)


```bash
sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
newgrp docker
sudo chmod 777 /var/run/docker.sock
```


![Alt Text](./assets/Screenshot%202024-03-05%20at%2010.53.09%20PM.png)

Build and run your application using Docker containers:

- ```docker build -t netflix .```


![Alt Text](./assets/Screenshot%202024-03-05%20at%2011.03.34%20PM.png)

Check your new image

- ``docker images``

![Alt Text](./assets/Screenshot%202024-03-05%20at%2011.13.10%20PM.png)


- ```docker run -d --name netflix -p 8081:80 netflix:latest```


![Alt Text](./assets/Screenshot%202024-03-05%20at%2011.24.00%20PM.png)


- with your local ip and port opening test if it works, you should notice it has no content


![Alt Text](./assets/Screenshot%202024-03-05%20at%2011.23.00%20PM.png)

- It will show an error cause you need API key

- to delete
- ``docker stop <containerid>``
- ``docker rmi -f netflix``

![Alt Text](./assets/Screenshot%202024-03-05%20at%2011.39.20%20PM.png)

## Step 5: 

- Get the API Key:

- Open a web browser and navigate to TMDB (The Movie Database) website.

- Click on "Login" and create an account.

- Once logged in, go to your profile and select "Settings."

- Click on "API" from the left-side panel.

- Create a new API key by clicking "Create" and accepting the terms and conditions.

- Provide the required basic details and click "Submit."

- You will receive your TMDB API key.

#### Now recreate the Docker image with your api key:

- docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .


![Alt Text](./assets/Screenshot%202024-03-05%20at%2011.43.53%20PM.png)


- run the latest image

- ```docker run -d -p 8081:80 netflix```


![Alt Text](./assets/Screenshot%202024-03-05%20at%2011.50.57%20PM.png)


- reload the localhost


![Alt Text](./assets/Screenshot%202024-03-05%20at%2011.49.24%20PM.png)


#### Phase 2: Security

- Install SonarQube and Trivy:


- Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.


![Alt Text](./assets/Screenshot%202024-03-06%20at%2012.12.13%20AM.png)


- sonarqube

`docker run -d --name sonar -p 9000:9000 sonarqube:lts-community`

![Alt Text](./assets/Screenshot%202024-03-06%20at%2012.23.03%20AM.png)

- To access:

- publicIP:9000 (by default username & password is admin)




- To install Trivy:

    ```
    sudo apt-get install wget apt-transport-https gnupg lsb-release
    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
    echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
    sudo apt-get update
    sudo apt-get install trivy        
    ```


![Alt Text](./assets/Screenshot%202024-03-06%20at%2012.32.36%20AM.png)


- ``trivy image <imageid>``

![Alt Text](./assets/Screenshot%202024-03-06%20at%2012.42.56%20AM.png)


#### Integrate SonarQube and Configure:
- Integrate SonarQube with your CI/CD pipeline.
- Configure SonarQube to analyze code for quality and security issues.

#### Phase 3: CI/CD Setup

- Install Jenkins for Automation:

- Install Jenkins on the EC2 instance to automate deployment: Install Java

    ```bash
    sudo apt update
    sudo apt install fontconfig openjdk-17-jre
    java -version
    openjdk version "17.0.8" 2023-07-18
    OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
    OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
    
    #jenkins
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
    ```


- `` sudo service jenkins status``
![Alt Text](./assets/Screenshot%202024-03-06%20at%201.14.40%20AM.png)


- Access Jenkins in a web browser using the public IP of your EC2 instance.
- public_Ip:8080

![Alt Text](./assets/Screenshot%202024-03-06%20at%201.22.58%20AM.png)


- Install Necessary Plugins in Jenkins:

- Goto Manage Jenkins →Plugins → Available Plugins →

#### Install below plugins

-  Eclipse Temurin Installer (Install without restart)

-  SonarQube Scanner (Install without restart)

-  NodeJs Plugin (Install Without restart)

-  Email Extension Plugin

#### Configure Java and Nodejs in Global Tool Configuration

- Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save

![Alt Text](./assets/Screenshot%202024-03-06%20at%2010.24.51%20AM.png)


![Alt Text](./assets/Screenshot%202024-03-06%20at%2010.27.30%20AM.png)


#### SonarQube
- Create the token

- Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this
After adding sonar token

![Alt Text](./assets/Screenshot%202024-03-06%20at%2010.32.19%20AM.png)

Click on Apply and Save

The Configure System option is used in Jenkins to configure different server
Global Tool Configuration is used to configure different tools that we install using Plugins
We will install a sonar scanner in the tools.


![Alt Text](./assets/Screenshot%202024-03-06%20at%2010.42.21%20AM.png)


![Alt Text](./assets/Screenshot%202024-03-06%20at%2011.00.03%20AM.png)


#### Create a Jenkins webhook

Configure CI/CD Pipeline in Jenkins:

- Create a CI/CD pipeline in Jenkins to automate your application deployment.



```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}
```

![Alt Text](./assets/Screenshot%202024-03-06%20at%202.16.05%20PM.png)

![Alt Text](./assets/Screenshot%202024-03-06%20at%202.35.35%20PM.png)


Certainly, here are the instructions without step numbers:

- Install Dependency-Check and Docker Tools in Jenkins

- Install Dependency-Check Plugin:

- Go to "Dashboard" in your Jenkins web interface.

- Navigate to "Manage Jenkins" → "Manage Plugins."

- Click on the "Available" tab and search for "OWASP Dependency-Check."

- Check the checkbox for "OWASP Dependency-Check" and click on the "Install without restart" button.
 Configure Dependency-Check Tool:

#### After installing the Dependency-Check plugin, you need to configure the tool.

- Go to "Dashboard" → "Manage Jenkins" → "Global Tool Configuration."

- Find the section for "OWASP Dependency-Check."

- Add the tool's name, e.g., "DP-Check."

- Save your settings.


![Alt Text](./assets/Screenshot%202024-03-06%20at%203.02.39%20PM.png)

#### Install Docker Tools and Docker Plugins:

- Go to "Dashboard" in your Jenkins web interface.

- Navigate to "Manage Jenkins" → "Manage Plugins."

- Click on the "Available" tab and search for "Docker."

- Check the following Docker-related plugins:

- Docker

- Docker Commons

- Docker Pipeline

- Docker API

- docker-build-step

Click on the "Install without restart" button to install these plugins.

#### Add DockerHub Credentials:

To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:

- Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."

- Click on "System" and then "Global credentials (unrestricted)."

- Click on "Add Credentials" on the left side.

- Choose "Secret text" as the kind of credentials.

- Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").

- Click "OK" to save your DockerHub credentials.


![Alt Text](./assets/Screenshot%202024-03-06%20at%202.57.30%20PM.png)

Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.

![Alt Text](./assets/Screenshot%202024-03-06%20at%203.14.20%20PM.png)

- stop and remove the netflix running image

![Alt Text](./assets/Screenshot%202024-03-06%20at%203.20.00%20PM.png)


#### Update Jenkins pipeline 

```groovy

pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=<yourapikey> -t netflix ."
                       sh "docker tag netflix nasi101/netflix:latest "
                       sh "docker push nasi101/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image nasi101/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 nasi101/netflix:latest'
            }
        }
    }
}


If you get docker login failed errorr

sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins


```