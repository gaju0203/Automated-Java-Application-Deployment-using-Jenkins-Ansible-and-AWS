 # 🚀 CI/CD Pipeline using Jenkins, Ansible, AWS S3 and Tomcat

This project demonstrates a real-time DevOps CI/CD pipeline where application code pushed to GitHub is automatically built using Jenkins, stored in AWS S3, and deployed to multiple Tomcat servers using Ansible.

The objective of this project is to simulate a production-style automated deployment workflow using industry-standard DevOps tools.

<img width="1536" height="1024" alt="ChatGPT Image Mar 6, 2026, 11_37_43 AM" src="https://github.com/user-attachments/assets/0416a262-c4db-457a-9383-75ad7740099b" />


## 📌 Project Architecture
Developers
     │
     ▼
GitHub Repository
     │
     ▼
Jenkins CI Server
(Checkout → Build → Test → Package)
     │
     ▼
Store Artifact (.war)
     │
     ▼
AWS S3 Bucket
     │
     ▼
Run Ansible Playbook
     │
 ┌───┴───────────────┐
 ▼       ▼        ▼
Tomcat1 Tomcat2 Tomcat3
(Worker Nodes)

<img width="1919" height="954" alt="Screenshot 2026-03-06 115032" src="https://github.com/user-attachments/assets/46d9389c-1daa-434f-9e02-0ee2f8766455" />

# ⚙️ Tools Used

-AWS EC2

-Jenkins

-Ansible

-Apache Tomcat

-Maven

-GitHub

-AWS S3

-Java Web Application

# 📂 Step 1: Source Code Repository
https://github.com/ReyazShaik/java-project-maven-new.git

# 🔐 Step 2: Enable Root Login & SSH Configuration
Set root password:
sudo passwd root

Edit SSH configuration file:
sudo vi /etc/ssh/ssh_config

Update the following lines.

Line 40
PermitRootLogin yes

Line 65
PasswordAuthentication yes

Restart SSH service:
sudo systemctl restart sshd
<img width="1919" height="892" alt="Screenshot 2026-03-06 121242" src="https://github.com/user-attachments/assets/f3b83a19-34e2-4093-92fd-80a6f837204b" />
<img width="1895" height="1018" alt="Screenshot 2026-03-06 121032" src="https://github.com/user-attachments/assets/1e0f96be-42b7-4251-89fe-b61a6168d5c7" />
<img width="1914" height="985" alt="Screenshot 2026-03-06 121108" src="https://github.com/user-attachments/assets/fdd6635a-076b-47ce-9201-a0f8218f9c1e" />

# ⚙️ Step 3: Install Jenkins and Ansible

Install required tools on Jenkins Server.

Update packages

sudo yum update -y

Install Java

sudo yum install java-17-amazon-corretto -y

Install Jenkins

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins -y

Start Jenkins

sudo systemctl start jenkins
sudo systemctl enable jenkins

Access Jenkins

http://SERVER-IP:8080

Install Ansible

sudo amazon-linux-extras install ansible2 -y

# 📦 Step 4: Install Required Jenkins Plugins
Manage Jenkins → Manage Plugins

Install plugins:

Ansible
Maven Integration
Deploy to Container
S3 Publisher
Git Plugin
Restart Jenkins.

# ☁️ Step 5: Configure AWS S3 for Artifact Storage
Manage Jenkins → System
Profile Name : s3creds
Access Key : XXXXX
Secret Key : XXXXX
Region : ap-south-1

# 🧱 Step 6: Jenkins Pipeline (Build & Upload Artifact)
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/ReyazShaik/java-project-maven-new.git'
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

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Upload to S3') {
            steps {
                s3Upload entries: [[
                bucket: 'jenkins-artifacts-demo',
                sourceFile: '**/*.war',
                selectedRegion: 'ap-south-1'
                ]],
                profileName: 's3creds'
            }
        }

    }
}

# This pipeline will:

Pull code from GitHub
Build Java application
Run tests
Generate WAR artifact
Upload artifact to AWS S3

# ⚙️ Step 7: Install Tomcat on Worker Nodes (Using Ansible)
vi tomcat.yml
---
---
- name: Install Tomcat
  hosts: all
  become: yes

  tasks:

    - name: Install Java
      yum:
        name: java-17-amazon-corretto
        state: present

    - name: Download Tomcat
      get_url:
        url: https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.35/bin/apache-tomcat-10.1.35.tar.gz
        dest: /root/

    - name: Extract Tomcat
      command: tar -zxvf apache-tomcat-10.1.35.tar.gz

    - name: Rename Tomcat
      command: mv apache-tomcat-10.1.35 tomcat

ansible-playbook tomcat.yml

# 🚀 Step 8: Deployment Playbook
vi deploy.yml

---
- name: Deploy WAR to Tomcat Servers
  hosts: all

  tasks:

    - name: Copy WAR file
      copy:
        src: /var/lib/jenkins/workspace/project/target/myapp.war
        dest: /root/tomcat/webapps/

# 🔄 Step 9: Jenkins Pipeline Deployment with Ansible
Add Ansible stage in pipeline.

stage('Deploy using Ansible') {
    steps {
        ansiblePlaybook credentialsId: 'linuxcreds',
        disableHostKeyChecking: true,
        installation: 'ansible',
        inventory: '/etc/ansible/hosts',
        playbook: '/etc/ansible/deploy.yml'
    }
}

# 🌐 Application Access
http://65.0.17..39:8080/myapp


<img width="1865" height="865" alt="Screenshot 2026-03-06 130711" src="https://github.com/user-attachments/assets/5c920485-3027-449c-b4da-2fd6005cb716" />

<img width="1893" height="920" alt="Screenshot 2026-03-06 130643" src="https://github.com/user-attachments/assets/7071944c-14a2-4562-8b0c-0bfc323450fa" />


