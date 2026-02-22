# 🚀 Node App Deployment & Automation with Jenkins

> A hands-on CI/CD project demonstrating automated deployment of a Node.js application using Jenkins Pipeline, SSH, PM2, and GitHub Webhooks.

---

## 📖 Project Overview

This project demonstrates how to automate the deployment of a Node.js application using **Pipeline as Code** in Jenkins.

Whenever code is pushed to the `main` branch on GitHub:

- Jenkins automatically clones the repository  
- Connects to a remote server via SSH  
- Copies the latest application files  
- Installs dependencies  
- Starts or restarts the app using PM2  

This setup simulates a real-world DevOps CI/CD workflow using two EC2 instances and secure SSH-based deployment.

---

## 🏗️ Architecture Overview

### CI/CD Flow

1. Developer pushes code to `main`
2. GitHub Webhook triggers Jenkins
3. Jenkins reads the `Jenkinsfile`
4. Jenkins connects to the Target Server via SSH
5. Files are deployed and managed using PM2

---

## 🖥️ Infrastructure Setup

This project uses **two EC2 instances**:

---

### 🔹 1️⃣ Jenkins Master Node

**Installed:**
- JDK
- Jenkins
- Git

**Required Plugins:**
- Git Plugin  
- SSH Agent Plugin  

**Security Group Ports:**
- 22 → SSH  
- 8080 → Jenkins UI  
- 80 → Optional  

---

### 🔹 2️⃣ Target Node (Application Server)

**Installed:**
- Node.js  
- npm  
- PM2  

**Security Group Ports:**
- 22 → SSH  
- 3000 → Node Application  
- 80 → HTTP  

Both instances allow SSH (Port 22).  
Master node additionally exposes Port 8080 for Jenkins.

---

## ⚙️ Installation Guide

---

### ☕ Install JDK (Master Node)

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

### 🛠 Install Jenkins

Follow the official installation guide:

👉 Jenkins: https://www.jenkins.io/doc/book/installing/

### 🚀 PM2 Installation Guide

#### 📦 Install PM2 Globally

```bash
sudo npm install -g pm2
```

## 🔐 SSH Credential Configuration (Using Existing Key)

Instead of generating a new SSH key, this project reuses the same private key already used to SSH into the EC2 instance from the local machine.

### Steps

1. Locate your existing key:

```bash
ls ~/.ssh
```

2. Copy contents of your private key (`id_rsa`).
3. In Jenkins:
   - Manage Jenkins
   - Credentials
   - Add Credentials
4. Set:
   - Kind → SSH Username with private key
   - ID → e.g., `ec2-ssh-key`
   - Username → `ubuntu`
   - Paste your existing private key

Now Jenkins authenticates using the same SSH identity you already use manually.

## 📂 Repository Structure

```text
node-app/
│
├── app.js
├── package.json
├── Jenkinsfile
└── README.md
```

Branch used: `main`

## 📜 Jenkins Pipeline - Core Logic (Sanitized Example)

### 🔹 Environment Configuration

```groovy
environment {
    SERVER_IP      = 'YOUR_TARGET_SERVER_IP'
    SSH_CREDENTIAL = 'your-ssh-credential-id'
    REPO_URL       = 'https://github.com/your-username/your-repo.git'
    BRANCH         = 'main'
    REMOTE_USER    = 'ubuntu'
    REMOTE_PATH    = '/home/ubuntu/node-app'
}
```

### 🔹 Clone Repository

```groovy
stage('Clone Repository') {
    steps {
        git branch: "${BRANCH}", url: "${REPO_URL}"
    }
}
```

### 🔹 Upload Files to Target Server

```groovy
stage('Upload Files to Target Server') {
    steps {
        sshagent([SSH_CREDENTIAL]) {
            sh """
                ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'mkdir -p ${REMOTE_PATH}'
                scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
            """
        }
    }
}
```

### 🔹 Install Dependencies & Start App

```groovy
sshagent([SSH_CREDENTIAL]) {
    sh """
        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
            cd ${REMOTE_PATH} &&
            npm install &&
            pm2 start app.js --name node-app || pm2 restart node-app
        '
    """
}
```

### 🔹 Post Build Status

```groovy
post {
    success {
        echo 'Application deployed successfully!'
    }
    failure {
        echo 'Deployment failed.'
    }
}
```

## 🔔 GitHub Webhook Configuration

In GitHub:

Repository → Settings → Webhooks → Add Webhook

Payload URL:

```text
http://YOUR_JENKINS_PUBLIC_IP:8080/github-webhook/
```

Now every push to `main` automatically triggers the pipeline.

## 🧠 What This Project Demonstrates

- CI/CD pipeline creation
- Pipeline as Code
- SSH-based remote deployment
- Credential management in Jenkins
- Process management with PM2
- Webhook-triggered automation
- Real-world DevOps workflow

## 🚀 Future Enhancements

- Convert into 3-Tier Architecture
- Add Nginx reverse proxy
- Dockerize application
- Add Load Balancer
- Implement Blue-Green deployment
- Add monitoring (Prometheus + Grafana)
- Deploy with Auto Scaling Groups

## 👨‍💻 Author

Prem Thakare
