# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm and Kubernetes

![Screenshot 2023-03-28 at 9 38 09 PM](https://user-images.githubusercontent.com/43399466/228301952-abc02ca2-9942-4a67-8293-f76647b6f9d8.png)


# 🚀 Jenkins Pipeline — Java App CI/CD
### Maven · SonarQube · Docker · Argo CD · Helm · Kubernetes

[![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://www.java.com/)
[![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)](https://maven.apache.org/)
[![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?style=for-the-badge&logo=sonarqube&logoColor=white)](https://www.sonarqube.org/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)](https://helm.sh/)
[![ArgoCD](https://img.shields.io/badge/Argo_CD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)

---

## 📖 Overview

This project demonstrates a **production-grade, end-to-end CI/CD pipeline** for a Java application.  
Every code push automatically travels through build → test → quality gate → containerization → GitOps-based deployment — **fully hands-free**.

```
Developer Push
      │
      ▼
┌─────────────────────────────────────────────────────┐
│                   JENKINS  (CI)                     │
│                                                     │
│  Git Checkout → Maven Build → Unit Tests            │
│       → SonarQube Scan → Docker Build & Push        │
│       → Update Helm Chart Image Tag in Git          │
└─────────────────────────┬───────────────────────────┘
                          │  Git commit triggers
                          ▼
┌─────────────────────────────────────────────────────┐
│                  ARGO CD  (CD / GitOps)             │
│                                                     │
│  Detect Git Change → Sync Helm Chart                │
│       → Deploy to Kubernetes → Health Check ✅      │
└─────────────────────────────────────────────────────┘
```

---

## 🗂️ Table of Contents

- [Pipeline Architecture](#-pipeline-architecture)
- [Prerequisites](#-prerequisites)
- [Step 1 — Jenkins Plugins](#step-1--install-jenkins-plugins)
- [Step 2 — Create Jenkins Pipeline](#step-2--create-jenkins-pipeline)
- [Step 3 — Pipeline Stages Overview](#step-3--pipeline-stages-overview)
- [Step 4 — Complete Jenkinsfile](#step-4--complete-jenkinsfile)
- [Step 5 — Configure SonarQube](#step-5--configure-sonarqube)
- [Step 6 — Set Up Argo CD](#step-6--set-up-argo-cd)
- [Step 7 — Helm Chart Structure](#step-7--helm-chart-structure)
- [Step 8 — Run the Pipeline](#step-8--run-the-pipeline)
- [Troubleshooting](#-troubleshooting)

---

## 🏗️ Pipeline Architecture

### Full 8-Stage CI/CD Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                          JENKINS PIPELINE                            │
├─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┤
│ Stage 1 │ Stage 2 │ Stage 3 │ Stage 4 │ Stage 5 │ Stage 6 │ Stage 7 │
│   Git   │  Maven  │  Unit   │ SonarQ  │ Docker  │  Helm   │  Argo   │
│Checkout │  Build  │  Tests  │  Scan   │  Push   │ Update  │CD Sync  │
│         │         │JUnit /  │Quality  │         │  Git    │(GitOps) │
│         │         │Mockito  │  Gate   │         │  Tag    │         │
└─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘
   ◄──────────────── CI (Jenkins) ─────────────────►◄─── CD (ArgoCD) ──►
```

### Tools Responsibility Map

| Stage | Tool | What It Does |
|-------|------|-------------|
| Source Control | Git + GitHub | Stores code; webhook triggers pipeline |
| Build | Maven | Compiles Java source, resolves dependencies |
| Unit Testing | JUnit + Mockito | Validates business logic in isolation |
| Code Quality | SonarQube | Detects bugs, vulnerabilities, code smells |
| Containerization | Docker | Packages the app into a portable image |
| Image Registry | DockerHub | Stores and versions Docker images |
| Helm Update | Git + Helm | Jenkins commits new image tag to `values.yaml` |
| GitOps Delivery | Argo CD | Auto-detects Git change → deploys to Kubernetes |

---

## ✅ Prerequisites

Before starting, ensure you have:

- [ ] Java application in a **GitHub repository**
- [ ] **Jenkins** server (EC2 `t2.medium` or larger recommended)
- [ ] **Kubernetes cluster** (EKS / Minikube / K3s)
- [ ] **Helm** installed on the Jenkins server
- [ ] **Argo CD** installed on the Kubernetes cluster
- [ ] **SonarQube** server running
- [ ] **DockerHub** account
- [ ] Jenkins server has internet access

---

## Step 1 — Install Jenkins Plugins

**Jenkins → Manage Jenkins → Manage Plugins → Available tab**

| Plugin | Purpose |
|--------|---------|
| `Git Plugin` | Clone source code from GitHub |
| `Maven Integration Plugin` | Build Java projects with Maven |
| `Pipeline Plugin` | Enable `Jenkinsfile` declarative pipelines |
| `Kubernetes Continuous Deploy Plugin` | Deploy to K8s via Helm |
| `SonarQube Scanner Plugin` | Run SonarQube analysis |
| `Docker Pipeline Plugin` | Build & push Docker images |
| `GitHub Integration Plugin` | Webhook trigger on push |

> 💡 After installing, restart Jenkins: `http://<jenkins-ip>:8080/restart`

---

## Step 2 — Create Jenkins Pipeline

### 2.1 Add GitHub Webhook

In your GitHub repo → **Settings → Webhooks → Add webhook**:
```
Payload URL : http://<jenkins-ip>:8080/github-webhook/
Content type: application/json
Trigger     : Just the push event ✅
```

### 2.2 Create Pipeline Job

1. **New Item** → Enter a name → Select **Pipeline** → OK
2. **Build Triggers** → ✅ `GitHub hook trigger for GITScm polling`
3. **Pipeline** → `Pipeline script from SCM`
4. **SCM** → `Git` → enter your repository URL
5. **Script Path** → `Jenkinsfile`
6. **Save**

### 2.3 Add Credentials to Jenkins

**Jenkins → Manage Jenkins → Credentials → Global → Add Credentials**

| Credential ID | Type | Used For |
|---------------|------|----------|
| `github` | Username + Token | Clone repos & push Helm tag updates |
| `dockerhub` | Username + Password | Push Docker images |
| `sonarqube-token` | Secret Text | SonarQube authentication |
| `argocd-token` | Secret Text | Argo CD API (optional) |

---

## Step 3 — Pipeline Stages Overview

| # | Stage | Key Actions |
|---|-------|-------------|
| 1 | **Checkout** | Clone the repo at the triggered commit SHA |
| 2 | **Maven Build** | `mvn clean package` — compiles all Java sources |
| 3 | **Unit Tests** | `mvn test` — runs JUnit/Mockito, publishes reports |
| 4 | **SonarQube Analysis** | Scans quality; fails build if quality gate not met |
| 5 | **Docker Build & Push** | Builds image tagged with `BUILD_NUMBER`, pushes to DockerHub |
| 6 | **Update Helm Chart** | Edits `values.yaml` image tag, commits back to Git |
| 7 | **Argo CD Sync** | Argo CD detects commit → syncs Helm chart → live on K8s |

---

## Step 4 — Complete Jenkinsfile

Add this file at the **root of your repository** as `Jenkinsfile`:

```groovy
pipeline {
    agent {
        docker {
            image 'abhishekf5/maven-abhishek-docker-agent:v1'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        SONAR_URL      = "http://<your-sonarqube-ip>:9000"
        DOCKER_IMAGE   = "your-dockerhub-username/java-app:${BUILD_NUMBER}"
        REGISTRY_CREDS = 'dockerhub'
        GIT_REPO_NAME  = "Jenkins-Zero-To-Hero"
        GIT_USER_NAME  = "your-github-username"
    }

    stages {

        // ─────────────────────────────────────
        // STAGE 1: Checkout Source Code
        // ─────────────────────────────────────
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/your-username/your-repo.git'
                echo '✅ Source code checked out'
            }
        }

        // ─────────────────────────────────────
        // STAGE 2: Build with Maven
        // ─────────────────────────────────────
        stage('Build') {
            steps {
                sh '''
                    cd java-maven-sonar-argocd-helm-k8s/spring-boot-app
                    mvn clean package
                '''
                echo '✅ Maven build complete'
            }
        }

        // ─────────────────────────────────────
        // STAGE 3: Unit Tests (JUnit + Mockito)
        // ─────────────────────────────────────
        stage('Unit Tests') {
            steps {
                sh '''
                    cd java-maven-sonar-argocd-helm-k8s/spring-boot-app
                    mvn test
                '''
            }
            post {
                always {
                    junit 'java-maven-sonar-argocd-helm-k8s/spring-boot-app/target/surefire-reports/*.xml'
                }
            }
        }

        // ─────────────────────────────────────
        // STAGE 4: SonarQube Code Quality Scan
        // ─────────────────────────────────────
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        cd java-maven-sonar-argocd-helm-k8s/spring-boot-app
                        mvn sonar:sonar \
                            -Dsonar.login=$SONAR_AUTH_TOKEN \
                            -Dsonar.host.url=${SONAR_URL}
                    '''
                }
                echo '✅ SonarQube scan complete'
            }
        }

        // ─────────────────────────────────────
        // STAGE 5: Docker Build & Push
        // ─────────────────────────────────────
        stage('Build & Push Docker Image') {
            steps {
                script {
                    sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', REGISTRY_CREDS) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
                echo "✅ Docker image pushed: ${DOCKER_IMAGE}"
            }
        }

        // ─────────────────────────────────────
        // STAGE 6: Update Helm Chart Image Tag
        // ─────────────────────────────────────
        stage('Update Helm Chart & Push to Git') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "jenkins@ci.com"
                        git config user.name "Jenkins CI"

                        # Patch the image tag in Helm values.yaml
                        sed -i "s/tag: .*/tag: ${BUILD_NUMBER}/" \
                            java-maven-sonar-argocd-helm-k8s/spring-boot-app-chart/values.yaml

                        git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-chart/values.yaml
                        git commit -m "ci: bump image tag to build-${BUILD_NUMBER} [skip ci]"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
                echo '✅ Helm chart updated — Argo CD will auto-sync shortly'
            }
        }

    } // end stages

    post {
        success {
            echo '🎉 Pipeline succeeded! Argo CD will deploy the new version to Kubernetes.'
        }
        failure {
            echo '❌ Pipeline failed. Check stage logs above.'
        }
        always {
            cleanWs()
        }
    }

} // end pipeline
```

> **Replace** `your-dockerhub-username`, `your-github-username`, `your-repo`, and `<your-sonarqube-ip>` with your actual values.

---

## Step 5 — Configure SonarQube

### 5.1 Run SonarQube via Docker (quickest)

```bash
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:lts-community
```

Access: `http://<your-server-ip>:9000`  
Default login: `admin` / `admin` (change on first login)

### 5.2 Generate Authentication Token

1. Log in → **My Account → Security → Generate Token**
2. Copy the token value
3. Add to Jenkins credentials with ID `sonarqube-token`

### 5.3 Configure SonarQube Server in Jenkins

**Jenkins → Manage Jenkins → Configure System → SonarQube Servers**

```
Name : sonarqube
URL  : http://<sonarqube-ip>:9000
Token: (select sonarqube-token credential)
```

### 5.4 Add SonarQube Properties to `pom.xml`

```xml
<properties>
    <sonar.projectKey>java-spring-app</sonar.projectKey>
    <sonar.projectName>Java Spring Boot App</sonar.projectName>
    <sonar.java.source>17</sonar.java.source>
</properties>
```

---

## Step 6 — Set Up Argo CD

### 6.1 Install Argo CD on Kubernetes

```bash
# Create dedicated namespace
kubectl create namespace argocd

# Install Argo CD
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Expose the UI externally
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'

# Get initial admin password
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

### 6.2 Create Argo CD Application Manifest

```yaml
# argocd-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: java-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/Jenkins-Zero-To-Hero.git
    targetRevision: main
    path: java-maven-sonar-argocd-helm-k8s/spring-boot-app-chart
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true       # Remove resources no longer in Git
      selfHeal: true    # Revert manual changes in the cluster
    syncOptions:
      - CreateNamespace=true
```

```bash
kubectl apply -f argocd-app.yaml
```

### 6.3 Add Argo CD API Token to Jenkins

```bash
# Login to Argo CD CLI
argocd login <argocd-server-ip>

# Generate a token for Jenkins
argocd account generate-token --account admin
```

Add the token to Jenkins credentials with ID `argocd-token`.

---

## Step 7 — Helm Chart Structure

```
spring-boot-app-chart/
├── Chart.yaml               # Chart name, version, description
├── values.yaml              # Default config — Jenkins updates image.tag here
├── templates/
│   ├── deployment.yaml      # K8s Deployment resource
│   ├── service.yaml         # K8s Service (ClusterIP / LoadBalancer)
│   ├── ingress.yaml         # Optional: external access via Ingress
│   └── _helpers.tpl         # Reusable template snippets
└── .helmignore
```

### `values.yaml` — Key Section Jenkins Updates

```yaml
image:
  repository: your-dockerhub-username/java-app
  tag: latest            # ← Jenkins CI updates this on every build
  pullPolicy: IfNotPresent

replicaCount: 2

service:
  type: ClusterIP
  port: 8080

resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "spring-boot-app.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "spring-boot-app.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "spring-boot-app.name" . }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 30
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 15
```

---

## Step 8 — Run the Pipeline

### 8.1 Trigger First Run Manually

1. **Jenkins → your pipeline job → Build Now**
2. Watch **Console Output** stage by stage
3. After success, verify:
   - ✅ **SonarQube** dashboard shows your project
   - ✅ **DockerHub** has the new image tagged with the build number
   - ✅ **GitHub** has a new commit updating `values.yaml`
   - ✅ **Argo CD** UI shows the app syncing/synced

### 8.2 Automatic Flow (after webhook)

```
git push origin main
      │
      ▼  (GitHub webhook fires instantly)
Jenkins pipeline starts
      │
      ▼  (Stages 1–6 complete in ~5–10 min)
New image tag committed to Git
      │
      ▼  (Argo CD polls Git every ~3 min)
Helm chart synced to Kubernetes cluster
      │
      ▼
🎉 New version is LIVE in production
```

### 8.3 Verify Deployment

```bash
# Check pods are running
kubectl get pods -n production

# Describe a pod for details
kubectl describe pod <pod-name> -n production

# View live app logs
kubectl logs -f deployment/java-app -n production

# Check the Helm release
helm list -n production

# Check Argo CD app status
argocd app get java-app
```

---

## 🔧 Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| `Permission denied` on Docker socket | Jenkins user not in docker group | `sudo usermod -aG docker jenkins && sudo systemctl restart docker` |
| SonarQube quality gate fails | Code issues exceed thresholds | Fix issues or adjust gate rules in SonarQube UI |
| `git push` fails in pipeline | GitHub token expired or wrong credential ID | Regenerate token → update Jenkins credential |
| Argo CD not syncing | Wrong Git path or app not configured | `argocd app get java-app` and check the `path` field |
| Pod in `CrashLoopBackOff` | App failing to start | `kubectl logs <pod-name> -n production` |
| Image not found on DockerHub | Wrong image name or missing tag | Verify `values.yaml` matches what was pushed |
| Pipeline not triggered by push | Webhook not configured | Check **GitHub → Settings → Webhooks** for delivery status |

---

## 📚 Learning Resources

- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Argo CD Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [Helm Chart Guide](https://helm.sh/docs/chart_template_guide/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [Kubernetes Docs](https://kubernetes.io/docs/home/)

---

## 📄 License

This project is licensed under the [MIT License](../LICENSE).

---

## 🙋‍♂️ Author

**S. Venkata Ramana**  
[![GitHub](https://img.shields.io/badge/GitHub-Ramana175-181717?style=flat&logo=github)](https://github.com/Ramana175)

---

> ⭐ If this helped you, please **star the repo** — it helps others find it too!

    7. Run the Jenkins pipeline:
       7.1 Trigger the Jenkins pipeline to start the CI/CD process for the Java application.
       7.2 Monitor the pipeline stages and fix any issues that arise.

This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD, Helm, and Kubernetes.
