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

- ``docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .``


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

![Alt Text](./assets/Screenshot%202024-03-06%20at%205.26.19%20PM.png)
![Alt Text](./assets/Screenshot%202024-03-06%20at%206.12.31%20PM.png)
![Alt Text](./assets/Screenshot%202024-03-06%20at%206.12.53%20PM.png)


#### Phase 4: Monitoring

- Install Prometheus and Grafana:

- Set up Prometheus and Grafana to monitor your application.

- Installing Prometheus:

- First, create a dedicated Linux user for Prometheus and download Prometheus:


![Alt Text](./assets/Screenshot%202024-03-06%20at%208.22.14%20PM.png)

```bash
sudo useradd --system --no-create-home --shell /bin/false prometheus
get https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
```

![Alt Text](./assets/Screenshot%202024-03-06%20at%209.14.32%20PM.png)



Extract Prometheus files, move them, and create directories:

   ```bash
   tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
   cd prometheus-2.47.1.linux-amd64/
   sudo mkdir -p /data /etc/prometheus
   sudo mv prometheus promtool /usr/local/bin/
   sudo mv consoles/ console_libraries/ /etc/prometheus/
   sudo mv prometheus.yml /etc/prometheus/prometheus.yml
   ```
![Alt Text](./assets/Screenshot%202024-03-06%20at%209.20.30%20PM.png)

   Set ownership for directories:

   ```bash
   sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
   ```

   Create a systemd unit configuration file for Prometheus:

   ```bash
   sudo nano /etc/systemd/system/prometheus.service
   ```

   Add the following content to the `prometheus.service` file:

   ```plaintext
   [Unit]
   Description=Prometheus
   Wants=network-online.target
   After=network-online.target

   StartLimitIntervalSec=500
   StartLimitBurst=5

   [Service]
   User=prometheus
   Group=prometheus
   Type=simple
   Restart=on-failure
   RestartSec=5s
   ExecStart=/usr/local/bin/prometheus \
     --config.file=/etc/prometheus/prometheus.yml \
     --storage.tsdb.path=/data \
     --web.console.templates=/etc/prometheus/consoles \
     --web.console.libraries=/etc/prometheus/console_libraries \
     --web.listen-address=0.0.0.0:9090 \
     --web.enable-lifecycle

   [Install]
   WantedBy=multi-user.target
   ```

   ![Alt Text](./assets/Screenshot%202024-03-06%20at%209.40.56%20PM.png)

#### Here's a brief explanation of the key parts in this `prometheus.service` file:

   - `User` and `Group` specify the Linux user and group under which Prometheus will run.

   - `ExecStart` is where you specify the Prometheus binary path, the location of the configuration file (`prometheus.yml`), the storage directory, and other settings.

   - `web.listen-address` configures Prometheus to listen on all network interfaces on port 9090.

   - `web.enable-lifecycle` allows for management of Prometheus through API calls.

   Enable and start Prometheus:

   ```bash
   sudo systemctl enable prometheus
   sudo systemctl start prometheus
   ```

   Verify Prometheus's status:

   ```bash
   sudo systemctl status prometheus
   ```

   ![Alt Text](./assets/Screenshot%202024-03-06%20at%209.51.14%20PM.png)

   Add inbound rule for port 9090

   ![Alt Text](./assets/Screenshot%202024-03-06%20at%209.58.12%20PM.png)

   You can access Prometheus in a web browser using your server's IP and port 9090:

   `http://<your-server-ip>:9090`

   ![Alt Text](./assets/Screenshot%202024-03-06%20at%2010.01.05%20PM.png)

   **Installing Node Exporter:**

   Create a system user for Node Exporter and download Node Exporter:

   ```bash
   sudo useradd --system --no-create-home --shell /bin/false node_exporter
   wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
   ```

   Extract Node Exporter files, move the binary, and clean up:

   ```bash
   tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
   sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
   rm -rf node_exporter*
   ```

   ![Alt Text](./assets/Screenshot%202024-03-06%20at%2011.43.27%20PM.png)

   Create a systemd unit configuration file for Node Exporter:

   ```bash
   sudo nano /etc/systemd/system/node_exporter.service
   ```

   Add the following content to the `node_exporter.service` file:

   ```plaintext
   [Unit]
   Description=Node Exporter
   Wants=network-online.target
   After=network-online.target

   StartLimitIntervalSec=500
   StartLimitBurst=5

   [Service]
   User=node_exporter
   Group=node_exporter
   Type=simple
   Restart=on-failure
   RestartSec=5s
   ExecStart=/usr/local/bin/node_exporter --collector.logind

   [Install]
   WantedBy=multi-user.target
   ```

   ![Alt Text](./assets/Screenshot%202024-03-06%20at%2011.49.22%20PM.png)

   Replace `--collector.logind` with any additional flags as needed.

   Enable and start Node Exporter:

   ```bash
   sudo systemctl enable node_exporter
   sudo systemctl start node_exporter
   ```

Verify the Node Exporter's status:

   ```bash
   sudo systemctl status node_exporter
   ```

   ![Alt Text](./assets/Screenshot%202024-03-06%20at%2011.53.40%20PM.png)

   You can access Node Exporter metrics in Prometheus.


## Configure Prometheus Plugin Integration

   Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

   **Prometheus Configuration:**

   To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the `prometheus.yml` file. Here is an example `prometheus.yml` configuration for your setup:

   ```yaml
   global:
     scrape_interval: 15s

   scrape_configs:
     - job_name: 'node_exporter'
       static_configs:
         - targets: ['localhost:9100']

     - job_name: 'jenkins'
       metrics_path: '/prometheus'
       static_configs:
         - targets: ['<your-jenkins-ip>:<your-jenkins-port>']
   ```

   Make sure to replace `<your-jenkins-ip>` and `<your-jenkins-port>` with the appropriate values for your Jenkins setup.

   Check the validity of the configuration file:

   ```bash
   promtool check config /etc/prometheus/prometheus.yml
   ```

   ![Alt Text](./assets/Screenshot%202024-03-07%20at%2012.39.20%20AM.png)

   Reload the Prometheus configuration without restarting:

   ```bash
   curl -X POST http://localhost:9090/-/reload
   ```

   You can access Prometheus targets at:

   `http://<your-prometheus-ip>:9090/targets`

   ![Alt Text](./assets/Screenshot%202024-03-07%20at%2012.44.43%20AM.png)


## Grafana

#### Install Grafana on Ubuntu 22.04 and Set it up to Work with Prometheus

#### Step 1: Install Dependencies:

First, ensure that all necessary dependencies are installed:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
```

#### Step 2: Add the GPG Key:

Add the GPG key for Grafana:

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

![Alt Text](./assets/Screenshot%202024-03-07%20at%2012.59.29%20AM.png)

#### Step 3: Add Grafana Repository:

Add the repository for Grafana stable releases:

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

![Alt Text](./assets/Screenshot%202024-03-07%20at%201.01.52%20AM.png)


#### Step 4: Update and Install Grafana:

Update the package list and install Grafana:

```bash
sudo apt-get update
sudo apt-get -y install grafana
```

#### Step 5: Enable and Start Grafana Service:

To automatically start Grafana after a reboot, enable the service:

```bash
sudo systemctl enable grafana-server
```

Then, start Grafana:

```bash
sudo systemctl start grafana-server
```


#### Step 6: Check Grafana Status:

Verify the status of the Grafana service to ensure it's running correctly:

```bash
sudo systemctl status grafana-server
```

![Alt Text](./assets/Screenshot%202024-03-07%20at%201.15.30%20AM.png)


#### Step 7: Access Grafana Web Interface:

Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. For example:

`http://<your-server-ip>:3000`

You'll be prompted to log in to Grafana. The default username is "admin," and the default password is also "admin."


#### Step 8: Change the Default Password:

When you log in for the first time, Grafana will prompt you to change the default password for security reasons. Follow the prompts to set a new password.

![Alt Text](./assets/Screenshot%202024-03-07%20at%201.23.10%20AM.png)

#### Step 9: Add Prometheus Data Source:

To visualize metrics, you need to add a data source. Follow these steps:

- Click on the gear icon (⚙️) in the left sidebar to open the "Configuration" menu.

- Select "Data Sources."

- Click on the "Add data source" button.

- Choose "Prometheus" as the data source type.

- In the "HTTP" section:
  - Set the "URL" to `http://localhost:9090` (assuming Prometheus is running on the same server).
  - Click the "Save & Test" button to ensure the data source is working.

  ![Alt Text](./assets/Screenshot%202024-03-07%20at%201.28.26%20AM.png)

#### Step 10: Import a Dashboard:

To make it easier to view metrics, you can import a pre-configured dashboard. Follow these steps:

- Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.

- Select "Dashboard."

- Click on the "Import" dashboard option.

- Enter the dashboard code you want to import (e.g., code 1860).
  ![Alt Text](./assets/Screenshot%202024-03-07%20at%201.30.40%20AM.png)

- Click the "Load" button.

- Select the data source you added (Prometheus) from the dropdown.

- Click on the "Import" button.

You should now have a Grafana dashboard set up to visualize metrics from Prometheus.
  ![Alt Text](./assets/Screenshot%202024-03-07%20at%201.36.08%20AM.png)

Grafana is a powerful tool for creating visualizations and dashboards, and you can further customize it to suit your specific monitoring needs.

That's it! You've successfully installed and set up Grafana to work with Prometheus for monitoring and visualization.


2. #### Configure Prometheus Plugin Integration:


- Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

- install prometheus in jenkins > manage jenkins > plugins > available plugins 
you will need to restart jenkins for the plugin to install successfully

![Alt Text](./assets/Screenshot%202024-03-07%20at%201.48.01%20AM.png)

- update the prometheus.yaml file and save .


![Alt Text](./assets/Screenshot%202024-03-07%20at%201.53.55%20AM.png)



refresh to see jenkins status <up>


![Alt Text](./assets/Screenshot%202024-03-07%20at%202.01.41%20AM.png)


- Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.

- Select "Dashboard."

- Click on the "Import" dashboard option.

- Enter the dashboard code you want to import (e.g., code 9964).


![Alt Text](./assets/Screenshot%202024-03-07%20at%202.07.03%20AM.png)


- Jenkins dashboard

![Alt Text](./assets/Screenshot%202024-03-07%20at%202.07.39%20AM.png)


#### Phase 5: Notification

1. #### Implement Notification Services:

- Set up email notifications in Jenkins or other notification mechanisms.


#### Phase 6: Kubernetes

## Create Kubernetes Cluster with Nodegroups

In this phase, you'll set up a Kubernetes cluster with node groups. This will provide a scalable environment to deploy and manage your applications.

## Monitor Kubernetes with Prometheus

Prometheus is a powerful monitoring and alerting toolkit, and you'll use it to monitor your Kubernetes cluster. Additionally, you'll install the node exporter using Helm to collect metrics from your cluster nodes.

### Install Node Exporter using Helm

To begin monitoring your Kubernetes cluster, you'll install the Prometheus Node Exporter. This component allows you to collect system-level metrics from your cluster nodes. Here are the steps to install the Node Exporter using Helm:

1. Add the Prometheus Community Helm repository:

    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    ```

2. Create a Kubernetes namespace for the Node Exporter:

    ```bash
    kubectl create namespace prometheus-node-exporter
    ```

3. Install the Node Exporter using Helm:

    ```bash
    helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter
    ```

Add a Job to Scrape Metrics on nodeip:9001/metrics in prometheus.yml:

Update your Prometheus configuration (prometheus.yml) to add a new job for scraping metrics from nodeip:9001/metrics. You can do this by adding the following configuration to your prometheus.yml file:


```
  - job_name: 'Netflix'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['node1Ip:9100']
```

Replace 'your-job-name' with a descriptive name for your job. The static_configs section specifies the targets to scrape metrics from, and in this case, it's set to nodeip:9001.

Don't forget to reload or restart Prometheus to apply these changes to your configuration.

To deploy an application with ArgoCD, you can follow these steps, which I'll outline in Markdown format:

### Deploy Application with ArgoCD

1. #### Install ArgoCD:

   You can install ArgoCD on your Kubernetes cluster by following the instructions provided in the [EKS Workshop](https://archive.eksworkshop.com/intermediate/290_argocd/install/) documentation.

2. #### Set Your GitHub Repository as a Source:

   After installing ArgoCD, you need to set up your GitHub repository as a source for your application deployment. This typically involves configuring the connection to your repository and defining the source for your ArgoCD application. The specific steps will depend on your setup and requirements.

3. #### Create an ArgoCD Application:
   - `name`: Set the name for your application.
   - `destination`: Define the destination where your application should be deployed.
   - `project`: Specify the project the application belongs to.
   - `source`: Set the source of your application, including the GitHub repository URL, revision, and the path to the application within the repository.
   - `syncPolicy`: Configure the sync policy, including automatic syncing, pruning, and self-healing.

4. #### Access your Application
   - To Access the app make sure port 30007 is open in your security group and then open a new tab paste your NodeIP:30007, your app should be running.

#### Phase 7: Cleanup

1. #### Cleanup AWS EC2 Instances:
    - Terminate AWS EC2 instances that are no longer needed.