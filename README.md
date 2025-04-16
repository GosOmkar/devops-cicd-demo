# CI/CD Case Study: Jenkins-based Pipeline to Deploy Dockerized Java App on EC2

## Objective

To build a CI/CD pipeline using Jenkins that:

- Pulls code from GitHub  
- Builds a Docker image for a Java application  
- Pushes the image to Docker Hub  
- Runs the container on an EC2 instance  

## Tech Stack

- Jenkins (CI/CD)  
- Docker (Containerization)  
- GitHub (Version Control)  
- AWS EC2 (Deployment)  
- Java (Sample Application)  

## Project Structure

```
devops-cicd-demo/
├── app/
│   └── HelloWorld.java
├── Dockerfile
├── Jenkinsfile
└── README.md
```

## Prerequisites

- GitHub repository containing:
  - Java application code
  - Dockerfile
  - Jenkinsfile
- AWS EC2 instance (Ubuntu)
- Jenkins installed on EC2
- Docker installed on EC2
- Docker Hub account

## Step-by-Step Implementation

### 1. Configure the EC2 Instance

Update the system and install essential packages:

```bash
sudo apt update && sudo apt install git curl -y
```

Install Docker:

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

Add Jenkins user to Docker group:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart docker
```

### 2. Install Jenkins on EC2

Add Jenkins repository:

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
```

Install Jenkins and Java:

```bash
sudo apt install openjdk-11-jdk jenkins -y
```

Start and enable Jenkins:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins via browser:

```
http://<EC2-Public-IP>:8080
```

### 3. GitHub Repository Setup

Clone or create a GitHub repository with the following structure:

Repository: https://github.com/GosOmkar/devops-cicd-demo

Contains:

- `Dockerfile`  
- `Jenkinsfile`  
- Java source file `HelloWorld.java` in the `app` directory

### 4. Jenkins Job Configuration

Create a new Jenkins pipeline:

- Navigate to Jenkins Dashboard → New Item → Pipeline  
- Configure GitHub URL: `https://github.com/GosOmkar/devops-cicd-demo.git`  
- Set Branch: `main`  
- Script Path: `Jenkinsfile`  
- Save and Build  

### 5. Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'your-dockerhub-username/java-hello-app'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/GosOmkar/devops-cicd-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh 'docker run -d -p 8081:8080 $IMAGE_NAME'
                }
            }
        }
    }
}
```

> Replace `your-dockerhub-username` with your actual Docker Hub username  
> Replace `dockerhub-creds` with the Jenkins credentials ID that holds your Docker Hub credentials

### 6. Validate the Deployment

Visit the deployed app in your browser:

```
http://<EC2-Public-IP>:8081
```

You should see your Java application running successfully.

---

## Key Learnings

- Built a complete CI/CD pipeline using Jenkins  
- Integrated GitHub with Jenkins to trigger builds  
- Used Docker to containerize a Java application  
- Managed build and deployment process using a Jenkinsfile  
- Secured Docker Hub credentials using Jenkins credentials  
- Deployed a containerized application on AWS EC2  

---

## Useful Resources

- Jenkins Installation Guide: https://www.jenkins.io/doc/book/installing/  
- Jenkins Pipeline Syntax: https://www.jenkins.io/doc/book/pipeline/syntax/  
- Docker Official Docs: https://docs.docker.com/get-started/  
- Docker Hub: https://hub.docker.com/  
- GitHub Webhooks: https://docs.github.com/en/webhooks  
