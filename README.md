Here is a **`README.md`** file for your **DEVSECOPS Project: Complete CI/CD (3-Tier App) - Petshop**. 

```markdown
# DEVSECOPS Project: Complete CI/CD (3-Tier App) - Petshop

## Overview
This project demonstrates a complete DevSecOps pipeline to deploy a Java-based **Petshop Application**, a common use case in organizations. The application is deployed using **Docker** and **Kubernetes (K8S)**, leveraging **Jenkins** as a CI/CD tool. This guide includes security integrations like SonarQube, OWASP Dependency Check, and Trivy for static and dynamic security scans.

### Features:
- Build and deploy using Docker Containers and Kubernetes clusters.
- Integrate with SonarQube for code analysis.
- Perform security scans using Trivy and OWASP Dependency Check.
- Automated deployment using Jenkins pipelines.
  
## Project Repository
[GitHub Repo](https://github.com/Aj7Ay/jpetstore-6.git)

---

## Prerequisites
- AWS Account for launching EC2 instances.
- Basic knowledge of Jenkins, Docker, Kubernetes, and Linux.

---

## Setup and Execution Steps

### 1. Create an AWS Ubuntu (22.04) EC2 Instance
- Launch a **T2 Large** instance.
- Enable **HTTP, HTTPS, and all ports** in the security group for learning purposes.

### 2. Install Jenkins, Docker, and Trivy
#### Install Jenkins:
Run the provided bash script to install Jenkins. Configure Jenkins to run on port **8090**.

#### Install Docker:
```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

#### Install Trivy:
Run the provided bash script to install Trivy.

#### Deploy SonarQube:
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
Configure SonarQube and retrieve the admin token.

---

### 3. Configure Jenkins
- Install plugins: **SonarQube Scanner**, **OWASP Dependency Check**, **Docker**, and **Pipeline** plugins.
- Configure global tools: **JDK 17**, **Maven**, and **SonarQube Scanner**.

---

### 4. Jenkins Pipeline Configuration
#### Sample Pipeline Script:
```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout SCM') {
            steps {
                git 'https://github.com/Aj7Ay/jpetstore-6.git'
            }
        }
        stage('Maven Compile and Test') {
            steps {
                sh 'mvn clean compile test'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petshop -Dsonar.java.binaries=. -Dsonar.projectKey=Petshop'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Build WAR File') {
            steps {
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build -t petshop .'
                        sh 'docker tag petshop <dockerhub-username>/petshop:latest'
                        sh 'docker push <dockerhub-username>/petshop:latest'
                    }
                }
            }
        }
        stage('TRIVY Security Scan') {
            steps {
                sh 'trivy image <dockerhub-username>/petshop:latest > trivy.txt'
            }
        }
        stage('Deploy to Docker Container') {
            steps {
                sh 'docker run -d --name pet1 -p 8080:8080 <dockerhub-username>/petshop:latest'
            }
        }
        stage('K8s Deployment') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
    }
    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}' - Jenkins Build #${env.BUILD_NUMBER}",
                body: "Build URL: ${env.BUILD_URL}<br/>Job: ${env.JOB_NAME}<br/>",
                to: 'postbox.aj99@gmail.com',
                attachmentsPattern: 'trivy.txt'
        }
    }
}
```

---

### 5. Kubernetes Setup
- Configure one instance as **Master** and another as **Worker**.
- Install Docker, `kubectl`, `kubeadm`, and `kubelet`.
- Initialize the master node and join the worker node.

---

### 6. Application Access
- **Docker Deployment**: Access the app at `<EC2-Public-IP>:8080/jpetstore`.
- **K8s Deployment**: Access via `<Node-IP>:<NodePort>`.

---

### 7. Terminate Instances
- Ensure all resources are terminated to avoid unnecessary costs.

---

## Contributors
- **Author**: [mrcloudbook.com](https://mrcloudbook.com)

## License
This project is licensed under the MIT License.
```

This README provides a structured overview, including instructions for setting up, configuring, and running your DevSecOps project. Adjust links or add images/screenshots for a polished presentation.
