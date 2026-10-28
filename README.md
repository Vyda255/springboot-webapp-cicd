# End-to-End CI/CD Pipeline Using Jenkins, Docker, SonarQube, ECR and AWS ECS

## 1. Project Overview

This project demonstrates the implementation of an end-to-end Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Java Spring Boot web application.

The application was developed using Spring Initializr and maintained in GitHub. An AWS EC2 instance running Ubuntu was configured as the CI/CD server, hosting Jenkins, Docker, and SonarQube.

Jenkins automates the application build, code quality analysis, Docker image creation, and publishing of the container image to Amazon Elastic Container Registry (ECR). The application is deployed on Amazon Elastic Container Service (ECS), with an Application Load Balancer (ALB) providing application access and Amazon CloudWatch providing infrastructure and service monitoring.

### Project Objectives

* Build a Spring Boot application and maintain its source code in GitHub.
* Set up a Jenkins CI/CD environment on AWS EC2.
* Integrate Jenkins with Docker and SonarQube.
* Automate application builds using Maven.
* Perform static code analysis using SonarQube.
* Containerize the application using Docker.
* Push Docker images to Amazon ECR.
* Deploy the application on Amazon ECS.
* Configure an Application Load Balancer.
* Monitor AWS resources using Amazon CloudWatch.

---
## 2. Technologies Used

| Technology                | Purpose                               |
| ------------------------- | ------------------------------------- |
| Java 17                   | Application development               |
| Spring Boot               | Web application framework             |
| Spring Initializr         | Spring Boot project generation        |
| Git and GitHub            | Source code management                |
| AWS EC2                   | Hosting the CI/CD tools               |
| Ubuntu                    | Operating system for the EC2 instance |
| Jenkins                   | CI/CD automation                      |
| Maven                     | Build and dependency management       |
| SonarQube                 | Static code analysis                  |
| Docker                    | Application containerization          |
| Amazon ECR                | Container image registry              |
| Amazon ECS                | Container deployment                  |
| Application Load Balancer | Application traffic routing           |
| Amazon CloudWatch         | AWS resource monitoring               |
| AWS IAM                   | Identity and access management        |

---

# 3. Implementation Steps

## Step 1: Create the Spring Boot Application

I created the Spring Boot project using Spring Initializr.

Configuration:

* Project: Maven
* Language: Java
* Java version: 17
* Packaging: Jar
* Build tool: Maven
* Dependency: spring web

I generated the project, downloaded the ZIP file, extracted it, and opened it in GitBash.

The application source code was maintained in the project directory.And I added Restcontroller to add a static message.

## Step 2: Run the Application Locally

Before implementing the CI/CD pipeline, I ran the application locally on Windows using Git Bash.

Command:

```bash

./mvnw spring-boot:run

```
This command uses the Maven Wrapper to start the Spring Boot application.


The application can be accessed locally at:

```text
http://localhost:8080
```

## Step 3: Push the Source Code to GitHub

I created a GitHub repository and pushed the Spring Boot application source code.

The repository contains the application code, Maven configuration, Dockerfile, Jenkinsfile, and project documentation.

Commands used for version control:

```bash
git init
git add .
git commit -m "Initial Spring Boot application"
git branch -M main
git remote add origin <your-github-repository-url>
git push -u origin main
```

GitHub serves as the source code repository for the CI/CD pipeline.

## Step 4: Create an AWS EC2 Instance

I created an EC2 virtual machine to host the CI/CD tools.

Configuration and setup included:

* Selected Ubuntu as the operating system.
* Selected an appropriate EC2 instance type.
* Configured the key pair for secure SSH access.
* Configured the security group to allow SSH on Port 22,JENKINS on Port 8080 and SonarQube on Port 9000.
* Used EBS volume of 28GB.
* Connected to the instance using SSH.

The EC2 instance was used to host Jenkins, Docker, and SonarQube.

### Security Group Configuration

The required inbound access was configured according to the services used.

Typical ports include:

| Port | Service   | Purpose                                           |
| ---- | --------- | ------------------------------------------------- |
| 22   | SSH       | Administrative access                             |
| 8080 | Jenkins   | Jenkins web interface, if configured on this port |
| 9000 | SonarQube | SonarQube web interface                           |

Port 8080 may also be used by the Spring Boot application in a different environment.


## Step 5: Install and Configure Jenkins

I installed Jenkins on the Ubuntu EC2 instance.

The Jenkins service was started and enabled to run after system restarts.

The Jenkins web interface was accessed through a browser using the EC2 instance public IP address and the configured Jenkins port.

Commands: 
1) //update the system and create java environment:
```bash
sudo apt update

sudo apt install fontconfig openjdk-21-jdk

java --version
```
2) 
Installing Jenkins
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install jenkins -y
```
3) Now,start and enable Jenkins using the following commands:
```bash
sudo systemctl enable jenkins

sudo systemctl start jenkins

sudo systemctl status jenkins      //shows active and running.
```
## Step 6: Install Docker

I installed Docker on the same EC2 instance using the following commands:
```bash
sudo apt install docker.io -y

sudo systemctl start docker  //starts docker

sudo systemctl enable docker //enables docker

docker --version  //checks docker version installed.
```

The Docker service was checked using:
```bash
sudo systemctl status docker
```
I also configured Jenkins to access the Docker daemon so that pipeline jobs could execute Docker commands,by adding jenkins to docker group:
```bash
sudo usermod -aG docker jenkins
```
Restart jenkins to apply the changes.


## Step 7: Install and Configure SonarQube

I installed SonarQube on the same Ubuntu EC2 instance by becoming a sonar user and following commands:
```bash
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-26.9.0.129388.zip  //downloads sonar binaries   which is a zip file

sudo unzip sonarqube-26.9.0.129388.zip //extract the zip file

sudo mv ~/sonarqube-26.9.0.129388 /opt/sonarqube

sudo chown -R sonar:sonar /opt/sonarqube

sudo systemctl daemon-reload

sudo systemctl start sonarqube
```
The SonarQube installation was configured as a service and started successfully.

SonarQube was accessed through its configured web interface.

Initial configuration included:

* Starting the SonarQube service.
* Verifying that the service was running.
* Accessing the SonarQube dashboard.
* Configuring Jenkins integration.
* Generating an authentication token for Jenkins integration.

The SonarQube server URL was configured in Jenkins using the EC2 private IP address and the configured SonarQube port.

The authentication token was stored in Jenkins Credentials rather than hardcoded into the Jenkinsfile.

SonarQube was used to analyze the application's source code for potential bugs, code smells, and maintainability issues.

## Step 8: Install AWS On The Same EC2 Instnace:
```bash
sudo snap install asw-cli --classic

aws configure //provide IAM User details
```
## Step 9: Install the Required Jenkins Plugins

I installed the Jenkins plugins needed for the CI/CD pipeline.

The relevant plugins included:

* Docker Pipeline
* SonarQube Scanner
* AWS Credentials
* Git plugin

After installation, Jenkins was configured to use the required tools and services.

### SonarQube Integration

I configured the SonarQube server in Jenkins system settings and stored the SonarQube authentication token in Jenkins Credentials.

The Jenkins pipeline used the SonarQube integration to submit the analysis.

### Docker Integration

The Docker Pipeline plugin and Docker daemon access enabled the pipeline to execute Docker-related stages.

## Step 10: Configure AWS Access for Jenkins

I configured AWS access for the pipeline by adding environment variables so it could authenticate with AWS and publish Docker images to Amazon ECR.

AWS access was configured using the available AWS authentication mechanism and appropriate permissions.


## Step 11: Create the Jenkinsfile

I created a `Jenkinsfile` to define the CI/CD pipeline as code.

The pipeline was designed to automate the application build, static code analysis, Docker image creation, and image publishing workflow.

The major pipeline stages were:

1. Checkout source code from GitHub.
2. Build and package the Spring Boot application using Maven.
3. Perform static code analysis using SonarQube.
4. Generate the application artifact.
5. Build the Docker image.
6. Authenticate with Amazon ECR.
7. Tag the Docker image.
8. Push the image to Amazon ECR.

The Jenkinsfile defined the execution steps and the required environment configuration for each stage.

## Step 12: Execute the Jenkins Pipeline

I configured a Jenkins pipeline job using the source code repository and Jenkinsfile.

When the pipeline was executed, Jenkins performed the configured stages.

The pipeline was used to validate the application build, submit code analysis to SonarQube, package the application into a Docker image, and publish the image to ECR.

Pipeline execution was verified through the Jenkins console output and build status.

## Step 13: Build the Application Using Maven

Maven was used to compile the application, execute the configured tests, and package the application into a JAR artifact.

The Maven build was executed as part of the Jenkins pipeline.

The resulting JAR artifact was then used to create the Docker image.

## Step 14: Perform Static Code Analysis

The pipeline integrated with SonarQube to analyze the application source code.

The analysis results were available in the SonarQube dashboard.

This integration demonstrated how code quality analysis can be incorporated into a CI/CD workflow.

## Step 15: Create the Docker Image

I created a Dockerfile to package the Spring Boot application artifact into a container image.

The Docker image included the application artifact and the required Java runtime environment.

The image was built as part of the Jenkins pipeline.

## Step 16: Push the Docker Image to Amazon ECR

I created an Amazon ECR repository named `springboot-app`.

The pipeline authenticated with ECR, tagged the Docker image, and pushed it to the repository.

The image was verified in the Amazon ECR console.

Amazon ECR served as the container image registry for the ECS deployment.

## Step 17: Deploy the Application to Amazon ECS

I configured Amazon ECS to run the containerized Spring Boot application using the image stored in ECR.

The ECS configuration included the resources needed to run the container, such as the cluster, task definition, and service.

The task definition specified the container image and the required runtime settings.

The ECS service managed the running tasks according to its configuration.

The application was deployed using the image published to Amazon ECR.

## Step 18: Configure the Application Load Balancer

I configured an Application Load Balancer to provide access to the deployed application.

The configuration included the relevant listener, target group, and ECS service integration.

The target group was configured with the appropriate application port and health check settings.

The load balancer routed incoming requests to healthy application targets.

The application was accessed through the load balancer DNS name after verifying that the targets were healthy.

## Step 19: Configure Amazon CloudWatch Monitoring

I used Amazon CloudWatch to inspect the metrics available for the configured AWS resources.

Monitoring included the available ECS service and Application Load Balancer metrics.

CloudWatch was used to review resource performance and deployment-related operational information.

Metric availability depends on the resource configuration, monitoring settings, and the selected time range.

---

# 4. Final CI/CD Workflow

The implemented workflow connects source code management, continuous integration, code quality analysis, containerization, and cloud deployment.

```text
Spring Initializr
       |
       v
Spring Boot Application
       |
       v
GitHub Repository
       |
       v
Jenkins Pipeline
       |
       v
Maven Build
       |
       v
SonarQube Analysis
       |
       v
Application JAR Artifact
       |
       v
Docker Image
       |
       v
Amazon ECR
       |
       v
Amazon ECS
       |
       v
Application Load Balancer
       |
       v
Spring Boot Application
```

Amazon CloudWatch monitors the configured AWS resources, including relevant ECS and load balancer metrics.


---

# 6. Key Learnings

Through this project, I gained practical experience in:

* Setting up an AWS EC2-based DevOps environment.
* Installing and configuring Jenkins, Docker, and SonarQube.
* Integrating tools into a Jenkins CI/CD pipeline.
* Building Java applications with Maven.
* Performing static code analysis.
* Containerizing applications with Docker.
* Publishing container images to Amazon ECR.
* Deploying containerized applications using Amazon ECS.
* Configuring an Application Load Balancer.
* Monitoring AWS resources with CloudWatch.
* Managing application source code and pipeline configuration using GitHub.

---

# 7. Conclusion

This project demonstrates the integration of Jenkins, Maven, SonarQube, Docker, and AWS services to establish a CI/CD workflow for a Spring Boot application.

It covers the process from application development and source code management to automated build, code analysis, container image publishing, cloud deployment, load balancing, and monitoring.

The project provided hands-on experience in implementing a cloud-based CI/CD workflow using DevOps tools and AWS container services.

---
