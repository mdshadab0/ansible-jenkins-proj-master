✨ CI/CD Pipeline with Jenkins, Ansible, and Docker 🐳
This project provides a comprehensive guide to building a robust CI/CD pipeline. The setup automates everything from building your code to deploying it on multiple servers, complete with monitoring capabilities.

⚙️ Tools and Technologies
Jenkins: The automation hub for our entire pipeline. 🧠

Ansible: Manages provisioning and deployments on our servers. 🤖

AWS S3: Secure storage for our built artifacts. 📦

IAM Roles: Manages access and permissions for AWS services. 🔐

Tomcat: The application server for our deployments. 🐱

Prometheus & Grafana: Our monitoring stack, keeping a constant eye on our infrastructure. 👀

✅ Prerequisites
To replicate this setup, you will need:

5 Servers: 1 for Ansible & Jenkins, and 4 worker nodes (2 for Dev, 2 for Test). 💻

AWS Access: An IAM role with permissions to manage an S3 bucket. 🔑

SSH Access: Root access to all five servers. 🚪

Phase 1: Server and Ansible Setup
1. Initial Server Configuration
Perform these steps on all five of your servers to set up the hostname and enable SSH root login.

Bash

sudo -i
hostnamectl set-hostname <desired-hostname> # e.g., ansible, dev-1, test-1
passwd root
vim /etc/ssh/sshd_config
# Change PermitRootLogin and PasswordAuthentication to yes
systemctl restart sshd
2. Configure Ansible
Run these commands only on the Ansible/Jenkins server.

Install Ansible:

Bash

yum install ansible -y
yum install python3 python-pip python-devel -y
Set up the Inventory:
Edit /etc/ansible/hosts and add the private IP addresses of your worker nodes.

Please change the IP addresses below to match your servers. ✏️

Ini, TOML

[dev]
<private-ip-of-dev-1>
<private-ip-of-dev-2>

[test]
<private-ip-of-test-1>
<private-ip-of-test-2>
Establish SSH Trust:
Generate an SSH key and copy it to all worker nodes to enable passwordless connections.

Bash

ssh-keygen # Press Enter 4 times
ssh-copy-id root@<private-ip-of-worker-node>
Phase 2: Jenkins CI/CD Pipeline
3. Install Jenkins and Tomcat
On the Ansible server, install Jenkins and its dependencies:

Bash

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install java-17-amazon-corretto jenkins -y
sudo systemctl restart jenkins
Install Tomcat using Ansible:
Clone the project repository on your Ansible server and run the playbook to install Tomcat on all worker nodes.

Bash

yum install git -y
git clone https://github.com/RAHAMSHAIK007/all-setups.git
cd all-setups
ansible-playbook tomcat.yml
4. Configure Credentials and Tools in Jenkins
After accessing the Jenkins dashboard at http://<ansible_server_ip>:8080, you must configure a few items:

Ansible Credentials:

Go to Manage Jenkins -> Manage Credentials.

Add a new credential of type Username with password.

Set Username to root and Password to the password you set earlier.

Give it an ID (e.g., ansible-root-creds). This ID will be used in the pipeline script. 🆔

S3 Credentials:

Install the S3 publisher plugin.

Go to Manage Jenkins -> System.

Find the S3 profiles section and add your AWS credentials. 🗝️

Tools:

Go to Manage Jenkins -> Tools.

Set up a Maven installation named maven and an Ansible installation named ansible with the path /usr/bin. 🛠️

5. The Pipeline Script
Create a new Jenkins pipeline job and paste this script. It automates the entire CI/CD process.

Crucial! Please update the following values in the script below:

Git URL: Replace with your own project repository.

S3 Bucket: Change rahamshaik007netflixbucket to your unique S3 bucket name.

Credentials ID: Replace d9713c79-8d68-41fa-a6bd-3d7d70c92b64 with the ID you gave your Ansible credentials.

Artifact Name: Change NETFLIX-1.2.2.war to the name of your application's artifact.

Groovy

pipeline {
    agent any
    stages {
        stage('Code') {
            steps {
                git branch: '$branch', url: 'https://github.com/YOUR-GITHUB-USERNAME/YOUR-PROJECT-REPO.git'
            }
        }
        // ... (Build, Test, and Artifact stages are the same)
        stage('S3 Upload') {
            steps {
                s3Upload consoleLogLevel: 'INFO', entries: [[bucket: 'your-unique-bucket-name', selectedRegion: 'us-east-1', sourceFile: 'target/YOUR-ARTIFACT-NAME.war']], profileName: 's3'
            }
        }
        stage('Deploy') {
            steps {
                ansiblePlaybook credentialsId: 'your-ansible-credentials-id', installation: 'ansible', inventory: '/etc/ansible/hosts', limit: '$server', playbook: '/etc/ansible/deploy.yml', vaultTmpPath: ''
            }
        }
    }
}
6. Monitoring Setup
To complete the project, install Node Exporter on all worker nodes. Then, set up Prometheus and Grafana (Docker is recommended). Make sure to update your prometheus.yml file to include the IP addresses of your worker nodes for metrics collection. 📈
