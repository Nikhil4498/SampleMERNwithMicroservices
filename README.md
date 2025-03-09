# MERN Microservices Deployment with AWS, Docker, Jenkins, Terraform, and Kubernetes

## Overview
This guide provides **detailed, step-by-step instructions** for deploying a **MERN Microservices-based application** using AWS services such as EC2, ECR, CodeCommit, Lambda, EKS, and Terraform. Additionally, it integrates **Jenkins** for CI/CD automation and **Helm** for Kubernetes deployment.

---
## 📌 Step 1: Set Up AWS Environment

### 1.1 Install AWS CLI & Boto3
AWS CLI and Boto3 are required to interact with AWS services.

#### **Install AWS CLI**
```sh
sudo apt update && sudo apt install -y awscli
aws --version  # Verify installation
```
#### **Configure AWS CLI**
```sh
aws configure
```
- Enter **AWS Access Key** & **Secret Key** (from AWS IAM User)
- Choose `us-east-1` as the default region

#### **Install Boto3 (Python SDK for AWS)**
```sh
pip install boto3
```

---
## 📌 Step 2: Prepare the MERN Application

### 2.1 Clone the Repository
```sh
git clone https://github.com/YourUsername/SampleMERNwithMicroservices.git
cd SampleMERNwithMicroservices
```

### 2.2 Containerize the Application with Docker

#### **Backend Dockerfile** (`backend/Dockerfile`)
```dockerfile
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "start"]
EXPOSE 5000
```

#### **Frontend Dockerfile** (`frontend/Dockerfile`)
```dockerfile
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "start"]
EXPOSE 3000
```

### 2.3 Build and Test Docker Images Locally
```sh
docker build -t backend-app ./backend
docker build -t frontend-app ./frontend
```

---
## 📌 Step 3: Push Docker Images to Amazon ECR

### 3.1 Create an ECR Repository
```sh
aws ecr create-repository --repository-name backend-app
aws ecr create-repository --repository-name frontend-app
```

### 3.2 Authenticate Docker with ECR
```sh
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com
```

### 3.3 Tag and Push Docker Images
```sh
docker tag backend-app:latest <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/backend-app:latest
docker push <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/backend-app:latest

docker tag frontend-app:latest <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/frontend-app:latest
docker push <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/frontend-app:latest
```

---
## 📌 Step 4: Version Control with AWS CodeCommit

### 4.1 Create a CodeCommit Repository
```sh
aws codecommit create-repository --repository-name MERN-Project
```
### 4.2 Push Code to CodeCommit
```sh
git remote add codecommit https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MERN-Project
git push codecommit main
```

---
## 📌 Step 5: Continuous Integration with Jenkins

### 5.1 Install Jenkins on EC2
```sh
sudo apt update && sudo apt install -y openjdk-11-jdk jenkins
sudo systemctl start jenkins && sudo systemctl enable jenkins
```

### 5.2 Configure Jenkins
- Access Jenkins: `http://<EC2-Public-IP>:8080`
- Install Plugins: **Pipeline, Docker, AWS Credentials**
- Create New Pipeline Job in Jenkins
- Add credentials for **AWS** and **GitHub**

### 5.3 Create a Jenkinsfile
#### **Jenkinsfile (CI/CD Pipeline)**
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/YourUsername/SampleMERNwithMicroservices.git'
            }
        }
        stage('Build Docker Images') {
            steps {
                sh 'docker build -t backend-app ./backend'
                sh 'docker build -t frontend-app ./frontend'
            }
        }
        stage('Push to ECR') {
            steps {
                sh 'docker push <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/backend-app:latest'
                sh 'docker push <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/frontend-app:latest'
            }
        }
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s-manifests/'
            }
        }
    }
}
```

---
## 📌 Step 6: Kubernetes Deployment with Helm on EKS

### 6.1 Install eksctl
```sh
curl -LO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz"
tar -xzf eksctl_Linux_amd64.tar.gz
sudo mv eksctl /usr/local/bin
```

### 6.2 Create an EKS Cluster
```sh
eksctl create cluster --name MERN-Cluster --region us-east-1
```

### 6.3 Deploy Application Using Helm
```sh
helm install mern-app ./helm-chart/
```

---
## 📌 Step 7: Monitoring & Logging

### 7.1 Enable CloudWatch Monitoring
```sh
aws cloudwatch put-metric-alarm --alarm-name "HighCPU" --metric-name "CPUUtilization" --namespace "AWS/EC2" --statistic Average --period 300 --threshold 80 --comparison-operator GreaterThanThreshold --dimensions Name=InstanceId,Value=<EC2-ID> --evaluation-periods 2 --alarm-actions <SNS-TOPIC-ARN>
```

### 7.2 View Logs
```sh
aws logs describe-log-groups
aws logs get-log-events --log-group-name "/ecs/backend-app" --log-stream-name "log-stream-name"
```

---
## ✅ Conclusion
By following these steps, we successfully:
- **Containerized the MERN application** using Docker.
- **Stored images** in Amazon ECR.
- **Implemented CI/CD pipeline** using Jenkins.
- **Deployed the application** on AWS EKS using Helm.
- **Monitored logs and metrics** using CloudWatch.

📌 **Your MERN Microservices Application is now deployed with AWS, Kubernetes, and CI/CD automation!** 🚀

