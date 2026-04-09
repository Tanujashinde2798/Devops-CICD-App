# DevOps CI/CD Pipeline with Jenkins, Docker & AWS ECR 🚀

This repository demonstrates a **real-world DevOps CI/CD pipeline** using **Jenkins**, **Docker**, and **AWS ECR**, deployed on a single EC2 instance.  
It’s designed for **learning purposes**, showing how code from GitHub can automatically be built, pushed to ECR, and deployed as a container.

---

## 🏗 Architecture

```
GitHub → Jenkins (EC2) → Docker → AWS ECR → EC2 (runs container)
```

- **Jenkins**: Installed on EC2, orchestrates the pipeline.
- **Docker**: Containerizes the application.
- **AWS ECR**: Stores Docker images for deployment.
- **EC2**: Hosts both Jenkins and the running container.

> ⚡ Simple, production-like setup for learning CI/CD without using EKS or Kubernetes.

---

## ⚙️ Requirements

- Ubuntu 22.04 EC2 instance
- Docker installed
- Jenkins installed
- AWS CLI configured with credentials
- AWS ECR repository

---

## 📝 Setup Instructions

### 1️⃣ Launch EC2

- OS: Ubuntu 22.04
- Instance type: t2.micro
- Storage: 20 GB
- Security Group: Open ports
  - 22 (SSH)
  - 8080 (Jenkins)
  - 80 (App)

---

### 2️⃣ Connect to EC2

```bash
ssh -i your-key.pem ubuntu@<your-ec2-public-ip>
```

---

### 3️⃣ Install Required Software

```bash
sudo apt update -y

# Docker
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu

# Java for Jenkins
sudo apt install openjdk-17-jdk -y

# Jenkins
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins

# AWS CLI
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Permissions
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

# Verify
docker --version
aws --version
```

---

### 4️⃣ Setup Jenkins UI

- Open: `http://<EC2-IP>:8080`
- Get initial password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Install suggested plugins
- Create admin user

---

### 5️⃣ Configure AWS CLI

```bash
aws configure
```

Enter:

- Access Key ID
- Secret Access Key
- Default region (e.g., `us-east-1`)

---

### 6️⃣ Create ECR Repository

```bash
aws ecr create-repository --repository-name devops-app
```

Copy repository URL:  
`<account-id>.dkr.ecr.<region>.amazonaws.com/devops-app`

---

### 7️⃣ Prepare GitHub Repository

Include:
- `Jenkinsfile`
- `Dockerfile`
- Application code (`app.js`, `package.json`)

---

### 8️⃣ Create Jenkins Pipeline

- Jenkins → New Item → Pipeline → Name: `devops-cicd`
- Pipeline script from SCM → Git → URL: your repo → Script Path: `Jenkinsfile`

---

### 9️⃣ Jenkinsfile Example

```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "<your-ecr-repo>"
        IMAGE_NAME = "devops-app"
        CONTAINER_NAME = "app"
    }

    stages {

        stage('Clone') {
            steps {
                git '<your-github-repo-url>'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh 'docker tag $IMAGE_NAME:latest $ECR_REPO:latest'
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $ECR_REPO:latest'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                docker run -d -p 80:3000 --name $CONTAINER_NAME $ECR_REPO:latest
                '''
            }
        }
    }
}
```

---

### 🔧 10️⃣ Run Pipeline

- Click **Build Now** in Jenkins
- Pipeline will:
  1. Pull code from GitHub
  2. Build Docker image
  3. Push image to ECR
  4. Deploy container on EC2

---

### 🌐 11️⃣ Test Deployment

Open in browser:  
`http://<EC2-IP>`  
You should see your application running.

---

### 🔁 12️⃣ Automate GitHub Pushes

- GitHub → Settings → Webhooks
- Payload URL: `http://<EC2-IP>:8080/github-webhook/`
- Content type: `application/json`
- Events: Just the push event

---

## 📸 Screenshots

1. Jenkins dashboard showing pipeline `devops-cicd`
   <div style="overflow:hidden; display:inline-block;">
  <img 
    src="https://github.com/Tanujashinde2798/Devops-CICD-App/raw/main/Screenshot%202026-04-09%20234907.png"
    width="500"
    style="transition: transform 0.3s ease;"
    onmouseover="this.style.transform='scale(1.1)'"
    onmouseout="this.style.transform='scale(1)'"
    alt="Jenkins Dashboard showing pipeline"
  />
</div>

2. Jenkins dashboard showing pipeline `devops-cicd`
    <div style="overflow:hidden; display:inline-block;">
  <img 
    src="https://github.com/Tanujashinde2798/Devops-CICD-App/raw/main/Screenshot%202026-04-09%20234842.png" 
    width="500"
    style="transition: transform 0.3s ease;"
    onmouseover="this.style.transform='scale(1.1)'"
    onmouseout="this.style.transform='scale(1)'"
  />
</div>

3. Build logs of a successful run
    <div style="overflow:hidden; display:inline-block;">
  <img 
    src="https://github.com/Tanujashinde2798/Devops-CICD-App/raw/main/Screenshot%202026-04-09%20235327.png"
    width="500"
    style="transition: transform 0.3s ease;"
    onmouseover="this.style.transform='scale(1.1)'"
    onmouseout="this.style.transform='scale(1)'"
    alt="Console Output Succesful Run"
  />
</div>

4. EC2 terminalon Mobaexterm showing `docker ps` with running container
    <div style="overflow:hidden; display:inline-block;">
  <img 
    src="https://github.com/Tanujashinde2798/Devops-CICD-App/raw/main/Screenshot%202026-04-10%20010453.png"
    width="500"
    style="transition: transform 0.3s ease;"
    onmouseover="this.style.transform='scale(1.1)'"
    onmouseout="this.style.transform='scale(1)'"
    alt="Docker ps on MobaXterm terminal"
  />
</div>

5. Browser showing app running at `http://<EC2-IP>`
    <div style="overflow:hidden; display:inline-block;">
  <img 
    src="https://github.com/Tanujashinde2798/Devops-CICD-App/raw/main/Screenshot%202026-04-09%20234842.png"
    width="500"
    style="transition: transform 0.3s ease;"
    onmouseover="this.style.transform='scale(1.1)'"
    onmouseout="this.style.transform='scale(1)'"
    alt="Jenkins Dashboard"
  />
</div>

6. AWS ECR console showing pushed image
    <div style="overflow:hidden; display:inline-block;">
  <img 
    src="https://github.com/Tanujashinde2798/Devops-CICD-App/raw/main/Screenshot%202026-04-09%20235029.png"
    width="500"
    style="transition: transform 0.3s ease;"
    onmouseover="this.style.transform='scale(1.1)'"
    onmouseout="this.style.transform='scale(1)'"
    alt="AWS ECR Registry"
  />
</div>

---

## 🗒 Notes

- This project is **for learning Jenkins and CI/CD pipelines**.
- No complex orchestration like EKS/Kubernetes — simplicity for understanding pipeline flow.
- Can be recreated on any EC2 for practice.

---

## 🎯 Key Learning Outcomes

- Jenkins pipeline creation & stages
- Docker image build and management
- AWS ECR integration
- Continuous deployment to EC2
- GitHub webhook triggers

---

