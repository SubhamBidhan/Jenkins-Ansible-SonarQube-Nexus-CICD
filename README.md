# 🔄 End-to-End CI/CD Pipeline — Jenkins + Maven + Ansible + SonarQube + Nexus + Tomcat on AWS

A production-grade, fully automated CI/CD pipeline that integrates **Jenkins, Maven, Ansible, SonarQube, Nexus, and Apache Tomcat** on AWS EC2 instances. The pipeline automatically pulls Java source code from GitHub, compiles and tests it, performs code quality analysis, stores the artifact, and deploys the WAR file to multiple Tomcat servers using Ansible playbooks.

---

## 📌 Project Overview

| Field | Details |
|---|---|
| **Type** | Personal Project |
| **Domain** | DevOps / CI-CD |
| **Duration** | 2026 |
| **Developer** | Subham Bidhan Sahoo |
| **Complexity** | High |

---

## 🏗️ Architecture Diagram

![CI/CD Architecture](architecture/cicd_architecture.png)

```
GitHub (java-maven-new)
        |
        | Code Push / Trigger
        ▼
Jenkins + Maven + Ansible Master (EC2 - t3.small)
        |
        |── stage: checkout  → git pull from GitHub
        |── stage: compile   → mvn compile
        |── stage: test      → mvn test
        |── stage: package   → mvn clean package → myapp.war
        |── stage: sonar     → SonarQube code analysis
        |── stage: nexus     → Upload WAR to Nexus Repository
        |── stage: deploy    → Ansible playbook → Copy WAR to Tomcat servers
        |
        |── SSH (Ansible) ──────────────────────────────────────┐
        |                                                        |
        ▼                           ▼                           ▼
  Tomcat Server 1            Tomcat Server 2            Tomcat Server 3
  (EC2 - t2.micro)           (EC2 - t2.micro)           (EC2 - t2.micro)
  Port: 8080                 Port: 8080                 Port: 8080

SonarQube Server (EC2 - t2.medium) → Port: 9000
Nexus Repository Server (EC2 - t2.medium) → Port: 8081
S3 Bucket → WAR file backup storage
```

---

## 🔧 Tech Stack

| Tool/Service | Purpose |
|---|---|
| **GitHub** | Source code repository |
| **Jenkins** | CI/CD orchestration and pipeline management |
| **Maven** | Java build tool — compile, test, package WAR |
| **Ansible** | Configuration management — Tomcat setup + WAR deployment |
| **Apache Tomcat** | Java application server for WAR deployment |
| **SonarQube** | Static code quality analysis and vulnerability detection |
| **Nexus Repository** | Artifact storage for WAR files (maven2 hosted) |
| **AWS S3** | WAR file backup storage |
| **AWS EC2** | Infrastructure for all servers |

---

## 📂 Project Structure

```
Jenkins-Ansible-CICD/
│
├── Jenkinsfile                     # Complete Jenkins pipeline (all stages)
├── ansible/
│   ├── tomcat.yml                  # Ansible playbook — Install & configure Tomcat
│   ├── deploy.yml                  # Ansible playbook — Deploy WAR to Tomcat servers
│   ├── tomcat-users.xml            # Tomcat user roles configuration
│   └── context.xml                 # Tomcat context configuration
│
├── installation/
│   ├── Ansible.sh
│   ├── Jenkins.sh
│   ├── Nexus.sh
│   └── Sonarqube.sh
|
├── architecture/
│   └── cicd_architecture.png       # Architecture diagram
│
└── README.md
```

---

## ⚙️ Infrastructure Setup

### EC2 Instances Required

| Instance | Type | Purpose |
|---|---|---|
| Jenkins + Ansible Master | t3.small | Jenkins server + Ansible control node |
| SonarQube Server | t2.medium | Code quality analysis |
| Nexus Repository Server | t2.medium | Artifact storage |
| Tomcat Server 1 | t2.micro | Application deployment target |
| Tomcat Server 2 | t2.micro | Application deployment target |
| Tomcat Server 3 | t2.micro | Application deployment target |

---

## 🚀 Setup & Integration Steps

### 🔌 Step 1 — Jenkins + Ansible Master
- Run [`installation/Jenkins.sh`](installation/Jenkins.sh) to install Jenkins → Access at `http://<IP>:8080`
- Install plugins: Git, Maven Integration, Ansible, SonarQube Scanner, Sonar Quality Gates, Nexus Artifact Uploader, S3 Publisher
- Run [`installation/Ansible.sh`](installation/Ansible.sh) to install Ansible + Python on the same instance
- Add Tomcat server IPs to `/etc/ansible/hosts` → Test with `ansible all -m ping`
- Configure Ansible path in Jenkins: **Tools → Ansible → Path = /bin**
- Add Tomcat SSH credentials: **Credentials → SSH Username with Private Key → Add PEM file**

---

### 🐱 Step 2 — Apache Tomcat on All Nodes
- Launch 3 EC2 instances for Tomcat deployment targets
- Run [`ansible/tomcat.yml`](ansible/tomcat.yml) to automatically install and configure Tomcat on all nodes
- Playbook handles: Tomcat download, Java 17 installation, `tomcat-users.xml`, `context.xml` configuration, and systemd service setup
- Verify: `http://<TOMCAT_IP>:8080`

---

### 🔍 Step 3 — SonarQube Integration
- Run [`installation/Sonarqube.sh`](installation/Sonarqube.sh) → Access at `http://<IP>:9000` (admin/admin)
- Create project → Generate token
- In Jenkins: Add SonarQube server URL + token under **System → SonarQube Servers**
- Pipeline uses `withSonarQubeEnv` to trigger scan automatically before artifact creation
- See [`Jenkinsfile`](Jenkinsfile) for the SonarQube stage

---

### 📦 Step 4 — Nexus Repository Integration
- Run [`installation/Nexus.sh`](installation/Nexus.sh) → Access at `http://<IP>:8081`
- Create a `maven2(hosted)` repository → Snapshot policy → Allow Redeploy
- In Jenkins: Add Nexus credentials → Configure `nexusArtifactUploader` in pipeline
- WAR file is automatically uploaded to Nexus after successful SonarQube analysis
- See [`Jenkinsfile`](Jenkinsfile) for the Nexus stage

---

### ☁️ Step 5 — S3 Artifact Backup
- Create a private S3 bucket for WAR file backup
- Configure AWS credentials in Jenkins: **System → Amazon S3 Profiles**
- Post-build action publishes `**/*.war` to S3 bucket automatically after pipeline run

---

### 🤖 Step 6 — Ansible Deployment Integration
- Ansible is integrated into Jenkins pipeline as a deploy stage
- Jenkins triggers [`ansible/deploy.yml`](ansible/deploy.yml) which copies the WAR file from Jenkins workspace to all Tomcat servers' webapps directory via SSH
- No manual login required to any Tomcat server
- See [`Jenkinsfile`](Jenkinsfile) for the Ansible deploy stage

---

### 🔄 Step 7 — Run the Pipeline
- Create a Pipeline job in Jenkins → Point to [`Jenkinsfile`](Jenkinsfile)
- Build triggers: checkout → compile → test → package → SonarQube → Nexus → Ansible Deploy
- Verify deployment: `http://<TOMCAT_IP>:8080/myapp`

---

## 🎯 Key Outcomes

- ✅ Fully automated CI/CD pipeline — code push to GitHub triggers entire pipeline automatically
- ✅ Automated code quality gate using SonarQube before artifact creation
- ✅ Centralized artifact management with Nexus Repository
- ✅ Zero-touch deployment to multiple Tomcat servers via Ansible playbooks
- ✅ WAR file backup stored in AWS S3
- ✅ Ansible eliminates manual SSH login to each server for deployment
- ✅ Tomcat runs as systemd service — auto-restarts on reboot

---

## 👨‍💻 Developer

**Subham Bidhan Sahoo**
- GitHub: [github.com/SubhamBidhan](https://github.com/SubhamBidhan)
- LinkedIn: [linkedin.com/in/subhambidhan](https://linkedin.com/in/subhambidhan)
