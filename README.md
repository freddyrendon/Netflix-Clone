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


- ```sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'```

- ```newgrp docker```

- ```sudo chmod 777 /var/run/docker.sock```


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

- ``sudo apt-get install wget apt-transport-https gnupg lsb-release``
- ``wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -``
- ``echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list``
- ``sudo apt-get update``
- ``sudo apt-get install trivy``


![Alt Text](./assets/Screenshot%202024-03-06%20at%2012.32.36%20AM.png)


- ``trivy image <imageid>``

![Alt Text](./assets/Screenshot%202024-03-06%20at%2012.42.56%20AM.png)


#### Integrate SonarQube and Configure:

- Integrate SonarQube with your CI/CD pipeline.

- Configure SonarQube to analyze code for quality and security issues.


