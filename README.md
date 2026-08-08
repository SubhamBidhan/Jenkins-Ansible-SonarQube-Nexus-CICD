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

## 🚀 Pipeline Stages
🔌 Step 1 — Jenkins + Ansible Master Setup
Launch an EC2 instance (Amazon Linux 2023, t3.small)
Run jenkins.sh to install Jenkins
Access Jenkins at http://<EC2_PUBLIC_IP>:8080
Install the following plugins via Manage Jenkins → Plugins:
Git Plugin
Maven Integration
Ansible Plugin
SonarQube Scanner
Sonar Quality Gates
Nexus Artifact Uploader
S3 Publisher
Install Ansible and Python on the same instance
Configure Ansible hosts file at /etc/ansible/hosts with Tomcat server IPs
Test connectivity: ansible all -m ping
Add Ansible path in Jenkins: Manage Jenkins → Tools → Ansible → Path = /bin
Add Tomcat SSH credentials in Jenkins: Manage Jenkins → Credentials → SSH Username with Private Key → Add PEM file
🐱 Step 2 — Apache Tomcat Setup on All Nodes
Launch 3 EC2 instances (Amazon Linux 2023, t2.micro) for Tomcat servers
Run the Ansible playbook to install Tomcat on all nodes automatically:
   ansible-playbook ansible/tomcat.yml

See ansible/tomcat.yml for the full playbook. This playbook will:

Download and extract latest Apache Tomcat
Install Java 17 (Amazon Corretto)
Configure tomcat-users.xml and context.xml
Create and enable Tomcat as a systemd service
Verify Tomcat is running: http://<TOMCAT_IP>:8080
🔍 Step 3 — SonarQube Setup
Launch an EC2 instance (Amazon Linux 2023, t2.medium — mandatory)
Run sonarqube.sh to install SonarQube
Start SonarQube manually:
   su - sonar
   sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start
Wait 2 minutes for startup, then access: http://<SONAR_IP>:9000
Default credentials: admin / admin
Create a new project manually: Add Project → Manually → Set Project Key → Generate Token
Integrate with Jenkins:
Add plugin: SonarQube Scanner, Maven Integration, Sonar Quality Gates
Add credentials: Manage Jenkins → Credentials → Secret Text → Paste Sonar Token
Configure server: Manage Jenkins → System → SonarQube Servers → Add URL + Token
Configure scanner: Manage Jenkins → Tools → SonarQube Scanner → Install Automatically
📦 Step 4 — Nexus Repository Setup
Launch an EC2 instance (Amazon Linux 2023, t2.medium — mandatory)
Run NexusAL2.sh to install Nexus
Access Nexus at: http://<NEXUS_IP>:8081
Default username: admin
Get password from: /nexus-data/admin.password
Create a repository:
Settings → Repositories → Create Repository → maven2 (hosted)
Name: hotstarapp
Version Policy: Snapshot
Deployment Policy: Allow Redeploy
Integrate with Jenkins:
Install plugin: Nexus Artifact Uploader
Add credentials: Manage Jenkins → Credentials → Username + Password → Nexus admin credentials
Update Nexus URL and repository name in Jenkinsfile
☁️ Step 5 — S3 Backup Setup
Create a private S3 bucket (e.g., subham-tomcat-artifacts)
Configure AWS credentials in Jenkins:
Manage Jenkins → System → Amazon S3 Profiles → Add Access Key + Secret Key
Add Post-build action in Jenkins job:
Publish artifacts to S3 bucket
Source: **/*.war
Destination: your S3 bucket name
Region: ap-south-1
🔄 Step 6 — Run the Pipeline
Create a new Pipeline job in Jenkins
Point it to the Jenkinsfile in this repository
Build the pipeline — it will automatically:
Pull code from GitHub
Compile and test using Maven
Analyze code quality with SonarQube
Upload WAR to Nexus
Deploy WAR to all Tomcat servers via Ansible
Verify deployment: http://<TOMCAT_IP>:8080/myapp

---

## 🎯 Key Outcomes

- ✅ Fully automated CI/CD pipeline — code push to GitHub triggers entire pipeline automatically
- ✅ Automated code quality gate using SonarQube before artifact creation
- ✅ Centralized artifact management with Nexus Repository
- ✅ Zero-touch deployment to multiple Tomcat servers via Ansible playbooks
- ✅ WAR file backup stored in AWS S3
- ✅ Ansible eliminates manual SSH login to each server for deployment
- ✅ PM2-equivalent reliability using systemd service for Tomcat auto-restart

---

## 🔌 Jenkins Plugins Required

- Git Plugin
- Maven Integration
- Ansible Plugin
- SonarQube Scanner
- Sonar Quality Gates
- Nexus Artifact Uploader
- S3 Publisher
- Pipeline: AWS Steps

---

## 👨‍💻 Developer

**Subham Bidhan Sahoo**
- GitHub: [github.com/SubhamBidhan](https://github.com/SubhamBidhan)
- LinkedIn: [linkedin.com/in/subhambidhan](https://linkedin.com/in/subhambidhan)
