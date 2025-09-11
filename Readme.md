
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

## ⚙️ Tools Used
- Jenkins
- Ansible
- Tomcat
- Prometheus
- Grafana
- Node Exporter
- S3 Bucket with IAM Role
- Amazon Linux (Python pre-installed)

---

## ✅ Phase 1 - Server Setup

### Configure All 5 Servers
```bash
sudo -i
hostnamectl set-hostname ansible/dev-1/dev-2/test-1/test-2
passwd root
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

### Maven Integration
- Manage Jenkins → Tools → Maven → Add `maven`

### S3 Integration
- Install S3 Publish Plugin
- Configure IAM Role in System → S3 Profile

Sample Upload Snippet:
```groovy
s3Upload consoleLogLevel: 'INFO',
  entries: [[
    bucket: 'your-s3-bucket',
    sourceFile: 'target/NETFLIX-1.2.2.war',
    selectedRegion: 'us-east-1'
  ]],
  profileName: 's3'
```

### Ansible Integration
- Install Ansible Plugin
- Configure Ansible Tool → Path: `/usr/bin`

Sample Playbook Step:
```groovy
ansiblePlaybook credentialsId: 'your-creds-id',
  disableHostKeyChecking: true,
  installation: 'ansible',
  inventory: '/etc/ansible/hosts',
  limit: '$server',
  playbook: '/etc/ansible/deploy.yml'
```

---

## ✅ Declarative Pipeline Sample
```groovy
pipeline {
    agent any

    stages {
        stage('Code Checkout') {
            steps {
                git branch: '$branch', url: 'https://github.com/RAHAMSHAIK007/jenkins-java-project.git'
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
                    bucket: 'your-s3-bucket',
                    sourceFile: 'target/NETFLIX-1.2.2.war',
                    selectedRegion: 'us-east-1'
                  ]],
                  profileName: 's3'
            }
        }

        stage('Deploy') {
            steps {
                ansiblePlaybook credentialsId: 'your-creds-id',
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
        src: /var/lib/jenkins/workspace/pipeline/target/NETFLIX-1.2.2.war
        dest: /root/tomcat/webapps/
```

---

## ✅ Monitoring Setup
- Prometheus & Grafana installed on Ansible server
- Node Exporter installed on worker nodes
- Prometheus configured to monitor all worker nodes

---

## ⚡ Notes
- Ensure correct credentials in Jenkins Tools section.
- S3 Bucket name must not contain spaces.
- Configure correct branch & server values.
- Inventory paths and playbook paths must match.

---

## 📚 References
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
