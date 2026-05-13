# 🚀 Jenkins Zero To Hero

> **A complete, hands-on Jenkins learning path** — from installation on AWS EC2 all the way to production-grade CI/CD pipelines with Docker, Kubernetes, ArgoCD, SonarQube, and Helm.

[![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)

---

## 📌 What You'll Learn

| Topic | Description |
|-------|-------------|
| ⚙️ Jenkins Setup | Install Jenkins on AWS EC2, configure Docker as a build agent |
| 🔁 Pipeline Basics | Write your first declarative pipeline from scratch |
| 🐳 Docker Integration | Build, tag, and push Docker images inside Jenkins |
| 🧪 Code Quality | Integrate SonarQube for static analysis |
| 📦 Helm + K8s | Deploy applications to Kubernetes using Helm charts |
| 🔄 GitOps with ArgoCD | Implement continuous deployment with ArgoCD |
| 🧩 Shared Libraries | Write reusable Jenkins pipeline code |
| 🤖 Multi-Agent Pipelines | Run stages in parallel across multiple agents |

---

## 📁 Repository Structure

```
Jenkins-Zero-To-Hero/
│
├── my-first-pipeline/                  # Beginner: Your first Jenkins declarative pipeline
│
├── multi-stage-multi-agent/            # Intermediate: Parallel stages with multiple agents
│
├── java-maven-sonar-argocd-helm-k8s/  # Advanced: Full CI/CD for Java app
│   ├── CI  →  Maven build + SonarQube scan + Docker push
│   └── CD  →  ArgoCD + Helm deploy to Kubernetes
│
├── python-jenkins-argocd-k8s/         # Advanced: Full CI/CD for Python app
│   ├── CI  →  Docker build + push
│   └── CD  →  ArgoCD deploy to Kubernetes
│
├── shared-libraries/                   # Reusable Groovy library functions
├── vars/                               # Global pipeline variables
│
├── Interview_Questions.md             # Jenkins interview Q&A for job prep
└── CODE_OF_CONDUCT.md
```

---

## 🛠️ Prerequisites

- AWS Account (Free Tier works)
- Basic Linux command-line knowledge
- Docker Hub account (for image registry)
- GitHub account

---

## ⚡ Quick Start — Jenkins on AWS EC2

### 1. Launch EC2 Instance
- Go to **AWS Console → EC2 → Launch Instance**
- Choose **Ubuntu 22.04 LTS**, instance type **t2.medium** (recommended)
- Open inbound port **8080** in the Security Group

### 2. Install Java & Jenkins

```bash
# Install Java
sudo apt update
sudo apt install openjdk-17-jre -y
java -version

# Install Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
```

### 3. Access Jenkins

```
http://<your-ec2-public-ip>:8080
```

Get the initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 4. Configure Docker as Build Agent

```bash
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
# Then restart Jenkins: http://<ip>:8080/restart
```

---

## 🔬 Project Deep Dives

### 📌 Project 1 — My First Pipeline
**Path:** `my-first-pipeline/`  
A simple declarative Jenkinsfile to understand pipeline syntax, stages, and steps.

### 📌 Project 2 — Multi-Stage Multi-Agent
**Path:** `multi-stage-multi-agent/`  
Learn how to run parallel stages on different agents to speed up your builds.

### 📌 Project 3 — Java App Full CI/CD
**Path:** `java-maven-sonar-argocd-helm-k8s/`

```
GitHub Push
    │
    ▼
Jenkins CI
    ├── Maven Build & Test
    ├── SonarQube Code Analysis
    ├── Docker Build & Push to DockerHub
    └── Update Helm chart image tag in Git
            │
            ▼
       ArgoCD CD
            └── Deploy to Kubernetes via Helm
```

### 📌 Project 4 — Python App Full CI/CD
**Path:** `python-jenkins-argocd-k8s/`  
Same GitOps pattern as above, adapted for Python applications.

### 📌 Shared Libraries
**Path:** `shared-libraries/` + `vars/`  
Write DRY (Don't Repeat Yourself) pipeline code by extracting common steps into reusable Groovy functions.

---

## 🧠 Tools & Technologies Used

| Tool | Purpose |
|------|---------|
| **Jenkins** | CI/CD orchestration |
| **Maven** | Java build tool |
| **SonarQube** | Code quality & security scanning |
| **Docker** | Containerization |
| **DockerHub** | Container image registry |
| **Kubernetes** | Container orchestration |
| **Helm** | Kubernetes package manager |
| **ArgoCD** | GitOps continuous delivery |
| **AWS EC2** | Cloud infrastructure |
| **GitHub** | Source code & GitOps repo |

---

## 📝 Jenkins Interview Questions

Check out [`Interview_Questions.md`](./Interview_Questions.md) for commonly asked Jenkins interview questions — great for DevOps job prep!

---

## 🤝 Contributing

Contributions are welcome! Feel free to open issues or submit pull requests to improve the examples or add new pipeline patterns.

---

## 📄 License

This project is licensed under the [MIT License](./LICENSE).

---

## 🙋‍♂️ Author

**S. Venkata Ramana**  
[![GitHub](https://img.shields.io/badge/GitHub-Ramana175-181717?style=flat&logo=github)](https://github.com/Ramana175)

---

> ⭐ If this repo helped you, please give it a star — it helps others find it too!
