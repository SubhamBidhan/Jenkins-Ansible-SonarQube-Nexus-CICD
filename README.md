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

### Complete Jenkinsfile
```groovy
pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/SubhamBidhan/java-project-maven-new.git'
            }
        }
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('package') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('Nexus-Artifact') {
            steps {
                nexusArtifactUploader artifacts: [[
                    artifactId: 'myapp',
                    classifier: '',
                    file: 'target/myapp.war',
                    type: '.war'
                ]],
                credentialsId: 'nexus',
                groupId: 'in.reyaz',
                nexusUrl: '<NEXUS_IP>:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'hotstarapp',
                version: '8.3.3-SNAPSHOT'
            }
        }
        stage('deploy') {
            steps {
                ansiblePlaybook(
                    playbook: '/etc/ansible/deploy.yml',
                    inventory: '/etc/ansible/hosts',
                    credentialsId: 'Linux_creds',
                    disableHostKeyChecking: true
                )
            }
        }
    }
}
```

---

## 📋 Ansible Playbooks

### tomcat.yml — Install & Configure Tomcat on All Nodes
```yaml
---
- name: Setup Tomcat
  hosts: all
  become: yes
  tasks:
    - name: Download Tomcat
      get_url:
        url: "https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.35/bin/apache-tomcat-10.1.35.tar.gz"
        dest: "/root/"

    - name: Extract Tomcat
      command: tar -zxvf apache-tomcat-10.1.35.tar.gz

    - name: Rename Tomcat directory
      command: mv apache-tomcat-10.1.35 tomcat

    - name: Install Java 17
      yum:
        name: java-17-amazon-corretto
        state: present

    - name: Configure tomcat-users.xml
      template:
        src: tomcat-users.xml
        dest: /root/tomcat/conf/tomcat-users.xml

    - name: Configure context.xml
      template:
        src: context.xml
        dest: /root/tomcat/webapps/manager/META-INF/context.xml

    - name: Create Tomcat systemd service
      copy:
        dest: /etc/systemd/system/tomcat.service
        content: |
          [Unit]
          Description=Apache Tomcat Server
          After=network.target
          [Service]
          User=root
          Group=root
          Type=forking
          Environment="JAVA_HOME=/usr/lib/jvm/jre"
          Environment="CATALINA_HOME=/root/tomcat"
          ExecStart=/root/tomcat/bin/startup.sh
          ExecStop=/root/tomcat/bin/shutdown.sh
          Restart=on-failure
          [Install]
          WantedBy=multi-user.target

    - name: Reload systemd and start Tomcat
      systemd:
        name: tomcat
        daemon_reload: yes
        state: started
        enabled: yes
```

### deploy.yml — Deploy WAR to All Tomcat Servers
```yaml
---
- name: Deploy WAR to Tomcat servers
  hosts: all
  tasks:
    - name: Copy WAR file to Tomcat webapps
      copy:
        src: /var/lib/jenkins/workspace/project/target/myapp.war
        dest: /root/tomcat/webapps
```

---

## 🔍 SonarQube Setup

```bash
# Start SonarQube manually (after EC2 restart)
su - sonar
sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start

# Access SonarQube
http://<SONAR_IP>:9000
# Default credentials: admin/admin
```

**Jenkins Integration:**
- Plugin: SonarQube Scanner, Maven Integration, Sonar Quality Gates
- System: Add SonarQube Server URL + authentication token
- Tools: Add SonarQube Scanner installation

---

## 📦 Nexus Repository Setup

```
# Access Nexus
http://<NEXUS_IP>:8081
# Default credentials: admin/<password from /nexus-data/admin.password>

Repository Type: maven2 (hosted)
Repository Name: hotstarapp
Version Policy: Snapshot
Deployment Policy: Allow Redeploy
```

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
