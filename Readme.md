
# 🚀 DeployFlow - CI/CD Pipeline Automation

## ✅ Project Overview
DeployFlow automates a full CI/CD pipeline using:
- **Jenkins** for Continuous Integration and Delivery
- **Ansible** for automation and configuration
- **Tomcat** for Java app deployment
- **Prometheus & Grafana** for server monitoring
- **S3 & IAM Role** for artifact storage

## 🚀 Architecture
- **1 Ansible Server (Jenkins Installed)**
- **4 Worker Nodes (2 DEV + 2 TEST)**  
All connected via SSH key authentication.

---

## ⚙️ Tools Used
- Jenkins
- Ansible
- Tomcat
- Prometheus
- Grafana
- Node Exporter
- S3 Bucket (with IAM Role for access)
- Amazon Linux (Python pre-installed)

---

## ✅ Phase 1 - Server Setup

### Configure All 5 Servers
```bash
sudo -i
hostnamectl set-hostname ansible/dev-1/dev-2/test-1/test-2
passwd root          # Set root password (used later in Ansible credentials)
vim /etc/ssh/sshd_config
# PermitRootLogin yes
# PasswordAuthentication yes
systemctl restart sshd
hostname -i
```

### Install Ansible (On Ansible Server)
```bash
yum install ansible -y
yum install python3 python-pip python-devel -y  # Optional

vim /etc/ansible/hosts
[dev]
172.31.20.40
172.31.21.25

[test]
172.31.31.77
172.31.22.114

ssh-keygen
ssh-copy-id root@<dev-1 IP>
ssh-copy-id root@<dev-2 IP>
ssh-copy-id root@<test-1 IP>
ssh-copy-id root@<test-2 IP>

ansible -m ping all
```

---

## ✅ Phase 2 - Jenkins & Pipeline Setup

### Install Jenkins
```bash
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum upgrade -y
yum install java-17-amazon-corretto jenkins -y
systemctl restart jenkins
systemctl daemon-reload
```

### Install Tomcat via Ansible
```bash
yum install git -y
git clone https://github.com/RAHAMSHAIK007/all-setups.git
cd all-setups
ansible-playbook tomcat.yml
```

---

## ✅ Jenkins Configuration

### 1️⃣ Maven Integration
- Manage Jenkins → Tools → Maven → Add name `maven`

---

### 2️⃣ S3 Integration
- Install S3 Publish Plugin
- Configure S3 Profile with your IAM Access Key and Secret Access Key  
⚠️ **Change this configuration based on your IAM user**  
Example:
```
IAM Access Key: YOUR_ACCESS_KEY
IAM Secret Key: YOUR_SECRET_KEY
Profile Name: s3
```

Pipeline Upload Snippet Example:
```groovy
s3Upload consoleLogLevel: 'INFO',
  entries: [[
    bucket: 'your-s3-bucket',       # ⚡ Change to your bucket name
    sourceFile: 'target/NETFLIX-1.2.2.war',
    selectedRegion: 'us-east-1'
  ]],
  profileName: 's3'
```

---

### 3️⃣ Ansible Integration
- Install Ansible Plugin
- Configure Ansible Tool → Path: `/usr/bin`

Ansible Playbook Execution Example:
```groovy
ansiblePlaybook credentialsId: 'your-creds-id',     # ⚡ Change to your Jenkins Credentials ID
  disableHostKeyChecking: true,
  installation: 'ansible',
  inventory: '/etc/ansible/hosts',
  limit: '$server',      # ⚡ Comes from choice parameter (dev/test)
  playbook: '/etc/ansible/deploy.yml'
```

⚡ **Important**:  
Provide correct **Ansible credential username & password (root)** when configuring Jenkins credentials.

---

## ✅ Declarative Jenkins Pipeline Sample

```groovy
pipeline {
    agent any

    stages {
        stage('Code Checkout') {
            steps {
                git branch: '$branch', url: 'https://github.com/RAHAMSHAIK007/jenkins-java-project.git'
                # ⚡ Change branch and URL as needed
            }
        }

        stage('Build') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Check') {
            steps {
                echo "Code quality checks passed"
            }
        }

        stage('Artifact Creation') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to S3') {
            steps {
                s3Upload consoleLogLevel: 'INFO',
                  entries: [[
                    bucket: 'your-s3-bucket',  # ⚡ Replace with your own bucket
                    sourceFile: 'target/NETFLIX-1.2.2.war',
                    selectedRegion: 'us-east-1'
                  ]],
                  profileName: 's3'
            }
        }

        stage('Deploy') {
            steps {
                ansiblePlaybook credentialsId: 'your-creds-id',   # ⚡ Replace with your Jenkins credentials ID
                  disableHostKeyChecking: true,
                  installation: 'ansible',
                  inventory: '/etc/ansible/hosts',
                  limit: '$server',
                  playbook: '/etc/ansible/deploy.yml'
            }
        }
    }
}
```

---

## ✅ Example deploy.yml

```yaml
- hosts: all
  tasks:
    - name: Copy WAR to Tomcat
      copy:
        src: /var/lib/jenkins/workspace/pipeline/target/NETFLIX-1.2.2.war    # ⚡ Ensure this matches your Jenkins workspace
        dest: /root/tomcat/webapps/
```

👉 **Tip**:  
If deployment fails, verify the `src` path matches your actual Jenkins WAR file location.

---

## ✅ Monitoring Setup
- Prometheus & Grafana installed on Ansible server
- Node Exporter installed on worker nodes
- Prometheus configured to monitor worker nodes

---

## ⚡ Final Notes
- Replace all placeholders:
    - IAM Access Key & Secret Access Key
    - S3 Bucket Name
    - Ansible Credentials (username: root and password)
    - Branch Name and GitHub URL
    - Jenkins Credentials ID
    - Inventory file paths
    - WAR file path in deploy.yml

---

## 📚 References
- [Jenkins Docs](https://www.jenkins.io/doc/)
- [Ansible Docs](https://docs.ansible.com/)
- [Prometheus Docs](https://prometheus.io/docs/)
- [Grafana Docs](https://grafana.com/docs/)
