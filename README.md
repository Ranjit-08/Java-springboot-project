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
