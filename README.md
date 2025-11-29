# 🚀 Full Stack Deployment: Spring Boot Backend + Streamlit Frontend (A–Z Guide)

This guide explains the complete process to build, package, and deploy a Spring Boot backend and a Streamlit frontend on an EC2 instance.
Includes: Maven build, running JAR, Python virtual environment, systemd service, and environment variables.

# 📦 1. Build & Package the Spring Boot Backend
connect the backend server 
 * sudo su -
 * yum install git -y
 * git clone https://github.com/CloudTechDevOps/Java-springboot-project.git
 * yum install maven -y
 * cd Java-springboot-project.git
   
Run the following command inside your backend project directory:

mvn clean package -Dspring.profiles.active=build


This generates a JAR file (example):

target/datastore-0.0.7.jar

📁 2. Move JAR File to /root Directory

Move your JAR file: 
go to target dir where the jar file exist 

mv datastore-0.0.7.jar /root

# 🔧 3. Run the Spring Boot Backend with MySQL RDS

Set the required environment variables and run the app using nohup:

MYSQL_HOST="jdbc:mysql://database-1.can0ws8oqjny.us-east-1.rds.amazonaws.com:3306/datastore?createDatabaseIfNotExist=true" \
MYSQL_USERNAME="admin" \
MYSQL_PASSWORD="Cloud123" \

nohup java -jar /root/datastore-0.0.7.jar > /var/log/app/nohup.out 2>&1 &

✔ Log directory
mkdir -p /var/log/app
chmod 777 /var/log/app

✔ Check backend service
ps -ef | grep java
tail -f /var/log/app/nohup.out

# 🎨 4. Install Python & Setup Virtual Environment for Frontend
* sudo su -
* yum install git -y
* git clone
* cd into your repo
* cd frontend 
Install Python 3 

sudo dnf install -y python3-pip


Create a virtual environment:

python3 -m venv venv


Activate it:

source venv/bin/activate


Upgrade pip:

pip install --upgrade pip


Install required packages:

pip install streamlit requests

🖥 5. Create frontend.service (Systemd Service)

This ensures Streamlit runs automatically in background and on server reboot.

✅ Option 1: If app.py is located directly in /root

Edit service file:

vi /etc/systemd/system/frontend.service


Paste:

[Unit]
Description=Streamlit Frontend App
After=network.target

[Service]
User=root
WorkingDirectory=/root
ExecStart=/root/venv/bin/streamlit run app.py --server.port=8501 --server.address=0.0.0.0
Environment=API_URL=http://172.31.25.254:8084
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

✅ Option 2: If app.py is inside a folder (recommended)

Example path:

/root/Java-springboot-project/frontend/app.py


Service file:

vi /etc/systemd/system/frontend.service


Paste:

[Unit]
Description=Streamlit Frontend App
After=network.target

[Service]
User=root
WorkingDirectory=/root/Java-springboot-project/frontend
ExecStart=/root/Java-springboot-project/frontend/venv/bin/python -m streamlit run /root/Java-springboot-project/frontend/app.py --server.port=8501 --server.address=0.0.0.0
Environment=API_URL=http://172.31.25.254:8084
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

# 🔥 Reload and Start the Service
systemctl daemon-reload
systemctl enable frontend
systemctl start frontend
systemctl status frontend

# 🔄 6. Managing Virtual Environment
Activate venv
source venv/bin/activate

Exit venv
deactivate

# 🚀 Access Your Frontend

Streamlit starts on port 8501:

http://<EC2-Public-IP>:8501


Backend runs on your custom port (example 8084):

http://<EC2-Public-IP>:8084

🎯 Summary of What You Have Done
Component	Technology	Deployment Method
Backend	Spring Boot	JAR running via nohup
Database	AWS RDS (MySQL)	JDBC URL + env variables
Frontend	Streamlit	systemd service
Virtual Environment	Python venv	Independent package setup
🎉 Completed!


✔ A fully deployed Spring Boot backend
✔ Running via nohup
✔ Connected to MySQL RDS
✔ A working Streamlit frontend
✔ Running as a Linux systemd service
✔ Auto-restaring on reboot
✔ API integrated using ENV variables

# Java Spring Boot Project with Jenkins & SonarQube Integration

This project demonstrates a full CI/CD pipeline using Jenkins and SonarQube for a Java Spring Boot backend and Python Streamlit frontend, deployed on AWS EC2 and connected to AWS RDS.
Table of Contents

-Project Overview
- Architecture
- AWS Setup
- EC2 Instance
- RDS Database
- Jenkins Installation
- SonarQube Installation
- Jenkins Configuration
- Pipeline Script
- Accessing Services

# Project Overview

This project automates:

Code Checkout: Pull from GitHub

Build: Maven clean & package

Code Quality: SonarQube analysis & Quality Gate

Deployment: Backend and Frontend automatically deployed

Monitoring: Logs captured in workspace
   
# EC2 Instance

Launch EC2 instance (Amazon Linux 2).

Connect via SSH:

ssh -i your-key.pem ec2-user@<EC2_PUBLIC_IP>
# Update and install Git:

sudo yum update -y
sudo yum install git -y

# RDS Database

Launch MySQL instance on RDS.

Database configuration:

Parameter	Value
Host	database-2.cwv0y4skwfjn.us-east-1.rds.amazonaws.com
Username	admin
Password	database-2

# Jenkins Installation
sudo yum install java-17-amazon-corretto.x86_64 -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins


Jenkins port: 8080

Access: http://<EC2_PUBLIC_IP>:8080

# SonarQube Installation
cd /opt/
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
sudo unzip sonarqube-8.9.6.50800.zip
sudo useradd sonar
sudo chown sonar:sonar sonarqube-8.9.6.50800 -R
sudo chmod 777 sonarqube-8.9.6.50800 -R
sudo passwd sonar
su - sonar
sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start
sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh status


SonarQube port: 9000

Login: admin/admin (change password after first login)

# Jenkins Configuration

Install plugins:

Pipeline Stage View

SonarQube Scanner

GitHub Integration

Add SonarQube credentials:

ID: sonarcred

Secret Text: <SONAR_TOKEN>

Configure SonarQube server in Jenkins:

Name: sonar

URL: http://<EC2_PUBLIC_IP>:9000/

Credentials: sonarcred

Create a Pipeline Job and paste the pipeline script below.

     pipeline {
       agent any

    environment {
        SONARQUBE = 'sonar'
        JAR_NAME = 'datastore-0.0.7.jar'
        BACKEND_PORT = '8084'
        FRONTEND_PORT = '8501'
        MYSQL_HOST = 'jdbc:mysql://database-2.cwv0y4skwfjn.us-east-1.rds.amazonaws.com:3306/datastore?createDatabaseIfNotExist=true'
        MYSQL_USERNAME = 'admin'
        MYSQL_PASSWORD = 'database-2'
        WORKSPACE_LOGS = "${WORKSPACE}/logs"
        PID_FILE = "${WORKSPACE}/backend.pid"
        BACKEND_WAIT_TIMEOUT = 120 // seconds
    }

    stages {

        stage('SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/CloudTechDevOps/Java-springboot-project.git'
            }
        }

        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Code Quality') {
            steps {
                withSonarQubeEnv(SONARQUBE) {
                    sh 'mvn clean package -Dspring.profiles.active=build'
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    echo "Quality Gate Response: ${qg}"
                    if (qg.status != 'OK') {
                        error "❌ Pipeline failed due to SonarQube Quality Gate."
                    } else {
                        echo "✅ Quality Gate Passed!"
                    }
                }
            }
        }

        stage('Build Package') {
            steps {
                sh 'mvn clean package -Dspring.profiles.active=build'
            }
        }

        stage('Prepare Workspace') {
            steps {
                sh """
                    mkdir -p ${WORKSPACE_LOGS}
                    cp target/${JAR_NAME} ${WORKSPACE}/${JAR_NAME}
                    chmod 755 ${WORKSPACE}/${JAR_NAME}
                """
            }
        }

        stage('Run Backend') {
            steps {
                sh """
                    # Stop previous backend if running
                    if [ -f ${PID_FILE} ]; then
                        kill -9 \$(cat ${PID_FILE}) || true
                    fi

                    # Start backend
                    nohup java -jar ${WORKSPACE}/${JAR_NAME} \\
                        -Dspring.profiles.active=build \\
                        -Dspring.datasource.url=${MYSQL_HOST} \\
                        -Dspring.datasource.username=${MYSQL_USERNAME} \\
                        -Dspring.datasource.password=${MYSQL_PASSWORD} \\
                        --server.port=${BACKEND_PORT} > ${WORKSPACE_LOGS}/backend.out 2>&1 &

                    echo \$! > ${PID_FILE}

                    # Wait for backend port to open with timeout
                    echo "Waiting for backend on port ${BACKEND_PORT}..."
                    elapsed=0
                    while ! lsof -i:${BACKEND_PORT} >/dev/null; do
                        sleep 5
                        elapsed=\$((elapsed+5))
                        echo "Backend not ready yet... waited \$elapsed seconds"
                        if [ \$elapsed -ge ${BACKEND_WAIT_TIMEOUT} ]; then
                            echo "❌ Backend did not start within ${BACKEND_WAIT_TIMEOUT} seconds. See logs:"
                            tail -n 20 ${WORKSPACE_LOGS}/backend.out
                            exit 1
                        fi
                    done
                    echo "✅ Backend is up!"
                """
            }
        }

        stage('Run Frontend') {
            steps {
                sh """
                    # Setup frontend venv if not exists
                    if [ ! -d frontend/venv ]; then
                        python3 -m venv frontend/venv
                        source frontend/venv/bin/activate
                        pip install --upgrade pip
                        pip install streamlit
                    else
                        source frontend/venv/bin/activate
                    fi

                    # Start Streamlit frontend
                    nohup streamlit run frontend/app.py --server.port=${FRONTEND_PORT} --server.address=0.0.0.0 > ${WORKSPACE_LOGS}/frontend.out 2>&1 &
                    echo "✅ Frontend started on port ${FRONTEND_PORT}"
                """
            }
        }

    }

    post {
        success {
            echo '🎉 Build, SonarQube check, backend & frontend deployment completed successfully.'
        }
        failure {
            echo '❌ Build or SonarQube Quality Gate validation failed. Check logs.'
        }
    }
    }
# Accessing Services
Service	URL
Jenkins	http://<EC2_PUBLIC_IP>:8080
SonarQube	http://<EC2_PUBLIC_IP>:9000
Backend	http://<EC2_PUBLIC_IP>:8084
Frontend	http://<EC2_PUBLIC_IP>:8501

# Troubleshooting

- Backend not starting → Check ${WORKSPACE}/logs/backend.out

- Frontend errors → Activate venv and ensure streamlit is installed

- SonarQube issues → Check logs /opt/sonarqube-8.9.6.50800/logs/
- 

