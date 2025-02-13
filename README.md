# Template-Deployment
## Overview
This project automates the deployment of the  template (a static website) on a Google Cloud Platform (GCP) Virtual Machine (VM) using a CI/CD pipeline with Jenkins, Docker, and Kubernetes. The Jenkins pipeline handles the build, push, and deployment process, while Kubernetes ensures the application's scalability and reliability.

## Prerequisites
Before setting up the project, ensure you have the following:

- A GCP account with billing enabled.
- A GCP Compute Engine VM instance (Ubuntu 22.04 recommended).
- Installed tools:
  - `git`
  - `docker`
  - `jenkins`
  - `kubectl`
  - `minikube`
## Setup Instructions

### 1. Clone the Repository
```sh
cd ~
git clone https://github.com/your-username/restoran-deployment.git
cd restoran-deployment
```

### 2. Set Up GCP VM Instance
- Create a Compute Engine VM instance on GCP with the following specifications:
  - OS: Ubuntu 22.04
  - Machine Type: `e2-medium` (2 vCPUs, 4 GB RAM)
  - Firewall: Allow HTTP and HTTPS traffic

- SSH into the VM:
```sh
gcloud compute ssh your-vm-instance-name --zone your-zone
```

### 3. Install Required Tools on the VM
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io git
```

#### Install Jenkins
```sh
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
echo "deb http://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update
sudo apt install -y openjdk-11-jdk jenkins
sudo systemctl enable --now jenkins
```

#### Install Kubernetes (kubectl and Minikube)
```sh
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start --driver=docker
```

### 4. Configure Jenkins
- Access Jenkins on `http://your-vm-external-ip:8080`.
- Install recommended plugins and configure Jenkins.
- Add credentials for Docker Hub and GitHub.
- Set up Jenkins pipeline (explained below).

## Repository Structure
```
restoran-deployment/
├── Dockerfile
├── jenkinsfile
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
├── src/
│   ├── index.html
│   ├── assets/
├── README.md
```

## Jenkins Pipeline
The `Jenkinsfile` automates the CI/CD process with the following stages:

1. **Clone Repository**: Pulls the latest code.
2. **Build Docker Image**: Builds an image using `nginx:alpine`.
3. **Push to Docker Hub**: Pushes the image to a Docker repository.
4. **Deploy to Kubernetes**: Applies Kubernetes configurations.

### Jenkinsfile Example
```groovy
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'your-dockerhub-username/restoran:latest'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-username/restoran-deployment.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f kubernetes/deployment.yaml'
                sh 'kubectl apply -f kubernetes/service.yaml'
            }
        }
    }
}
```

## Kubernetes Deployment

### Deployment YAML (`deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: restoran
spec:
  replicas: 2
  selector:
    matchLabels:
      app: restoran
  template:
    metadata:
      labels:
        app: restoran
    spec:
      containers:
      - name: restoran
        image: your-dockerhub-username/restoran:latest
        ports:
        - containerPort: 80
```

### Service YAML (`service.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: restoran-service
spec:
  selector:
    app: restoran
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

## Access the Application
- Get the external IP of the service:
```sh
kubectl get svc restoran-service
```
- Open the IP in your browser to access the website.

## Clean Up
To delete Kubernetes resources:
```sh
kubectl delete -f kubernetes/deployment.yaml
kubectl delete -f kubernetes/service.yaml
```
To stop Minikube:
```sh
minikube stop


