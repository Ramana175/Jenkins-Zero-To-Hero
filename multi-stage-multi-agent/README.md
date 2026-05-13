# Multi-Stage Multi-Agent Jenkins Pipeline

## Overview

This project demonstrates how to set up a **multi-stage Jenkins pipeline** where each stage runs on a **dedicated Docker-based agent**. Instead of using a single agent for the entire pipeline, each stage spins up its own isolated container, ensuring clean, reproducible environments for every part of your CI/CD workflow.

---

## Pipeline Architecture

```
pipeline (agent: none)
│
├── Stage: Back-end
│   └── Agent: Docker → maven:3.8.1-adoptopenjdk-11
│       └── Step: mvn --version
│
└── Stage: Front-end
    └── Agent: Docker → node:16-alpine
        └── Step: node --version
```

---

## Jenkinsfile

```groovy
pipeline {
    agent none
    stages {
        stage('Back-end') {
            agent {
                docker { image 'maven:3.8.1-adoptopenjdk-11' }
            }
            steps {
                sh 'mvn --version'
            }
        }
        stage('Front-end') {
            agent {
                docker { image 'node:16-alpine' }
            }
            steps {
                sh 'node --version'
            }
        }
    }
}
```

---

## Key Concepts

### `agent none` at the Top Level
Setting `agent none` at the pipeline level means **no global agent is allocated**. Each stage must declare its own agent. This is the foundation of a multi-agent pipeline.

### Stage-Level Agents
Each stage defines its own Docker image as the agent:
- **Back-end stage** uses `maven:3.8.1-adoptopenjdk-11` — a Java + Maven environment.
- **Front-end stage** uses `node:16-alpine` — a lightweight Node.js environment.

### Benefits
- **Isolation** — Each stage runs in a fresh container; no dependency conflicts between stages.
- **Flexibility** — Different stages can use completely different technology stacks.
- **Reproducibility** — Docker images guarantee consistent environments across builds.
- **Parallelism** — Stages can be configured to run in parallel on separate agents.

---

## Prerequisites

Before running this pipeline, ensure the following are set up:

| Requirement | Details |
|---|---|
| Jenkins | Version 2.x or higher |
| Docker | Installed on the Jenkins agent/node |
| Docker Pipeline Plugin | Install via Jenkins → Manage Plugins |
| Jenkins Docker access | Jenkins user must have permission to run Docker commands |

### Installing the Docker Pipeline Plugin

1. Go to **Jenkins Dashboard** → **Manage Jenkins** → **Manage Plugins**
2. Search for `Docker Pipeline`
3. Install and restart Jenkins

---

## Setup Instructions

### 1. Configure Jenkins with Docker as Agent

Ensure Docker is installed on your Jenkins server:

```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### 2. Create a New Pipeline Job

1. Go to **Jenkins Dashboard** → **New Item**
2. Enter a name (e.g., `multi-stage-multi-agent`)
3. Select **Pipeline** and click **OK**

### 3. Configure the Pipeline

Under the **Pipeline** section, either:

**Option A — Use the Jenkinsfile directly:**
- Select **Pipeline script**
- Paste the Jenkinsfile content

**Option B — Pull from SCM:**
- Select **Pipeline script from SCM**
- Set SCM to **Git**
- Enter repository URL: `https://github.com/Ramana175/Jenkins-Zero-To-Hero`
- Set the script path to: `multi-stage-multi-agent/Jenkinsfile`

### 4. Run the Pipeline

Click **Build Now** and monitor the execution in the **Stage View**.

---

## Expected Output

```
[Pipeline] stage
[Pipeline] { (Back-end)
[Pipeline] sh
+ mvn --version
Apache Maven 3.8.1 (...)
Java version: 11.x.x
...

[Pipeline] stage
[Pipeline] { (Front-end)
[Pipeline] sh
+ node --version
v16.x.x
...
```

---

## Extending the Pipeline

You can extend this pattern to build real-world CI/CD pipelines. Here's an example with additional stages:

```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker { image 'maven:3.8.1-adoptopenjdk-11' }
            }
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            agent {
                docker { image 'maven:3.8.1-adoptopenjdk-11' }
            }
            steps {
                sh 'mvn test'
            }
        }
        stage('Code Analysis') {
            agent {
                docker { image 'sonarsource/sonar-scanner-cli' }
            }
            steps {
                sh 'sonar-scanner'
            }
        }
        stage('Frontend Build') {
            agent {
                docker { image 'node:16-alpine' }
            }
            steps {
                sh 'npm install && npm run build'
            }
        }
        stage('Docker Build & Push') {
            agent any
            steps {
                sh 'docker build -t myapp:latest .'
                sh 'docker push myapp:latest'
            }
        }
    }
}
```

---

## Related Examples in This Repository

| Folder | Description |
|---|---|
| `my-first-pipeline` | Single-agent pipeline using Node.js Docker container |
| `multi-stage-multi-agent` | *(This example)* Multi-stage pipeline with per-stage Docker agents |
| `java-maven-sonar-argocd-helm-k8s` | Full CI/CD: Build → Test → SonarQube → Docker → Argo CD → Kubernetes |
| `python-jenkins-argocd-k8s` | Python app CI/CD with Argo CD and Kubernetes deployment |

---

## Troubleshooting

**Docker permission denied error**
```
Got permission denied while trying to connect to the Docker daemon socket
```
Fix: Add Jenkins user to the docker group and restart:
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

**Image pull failures**
Ensure the Jenkins node has internet access to pull images from Docker Hub, or configure a private registry mirror.

**Plugin not found**
Install the **Docker Pipeline** plugin from Jenkins Plugin Manager.

---

## References

- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Jenkins Docker Pipeline Plugin](https://plugins.jenkins.io/docker-workflow/)
- [Docker Hub — Maven Image](https://hub.docker.com/_/maven)
- [Docker Hub — Node.js Image](https://hub.docker.com/_/node)
