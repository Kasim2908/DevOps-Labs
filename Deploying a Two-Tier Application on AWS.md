# 🚀 Deploying a Two-Tier Application on AWS

A complete hands-on project demonstrating how to deploy a **Two-Tier Employee Management Application on AWS** using **Amazon VPC, EC2, Docker, Nginx, Application Load Balancer, Auto Scaling, and Amazon RDS MySQL**.

This project focuses on building a scalable and highly available cloud architecture while gaining practical experience with **AWS networking, containerization, database connectivity, load balancing, auto scaling, security groups, DNS, and real-world troubleshooting**.

---

## 📌 Project Overview

The application consists of two main tiers:

### 🖥️ Application Tier

* Amazon EC2
* Docker
* Nginx
* Flask Backend
* HTML/CSS/JavaScript Frontend
* Application Load Balancer
* Auto Scaling Group

### 🗄️ Database Tier

* Amazon RDS MySQL
* Private Subnets
* Database Security Group

### 🌐 Networking

* Amazon VPC
* Public Subnets
* Private Subnets
* Internet Gateway
* Route Tables
* Security Groups
* Multiple Availability Zones

---

## 🏗️ Architecture

```text
                         ┌──────────────────────┐
                         │       INTERNET       │
                         │        USERS         │
                         └──────────┬───────────┘
                                    │
                                    ▼
                       ┌────────────────────────┐
                       │ Application Load       │
                       │ Balancer (ALB)         │
                       │ HTTP :80               │
                       └────────────┬───────────┘
                                    │
                     ┌──────────────┴──────────────┐
                     │                             │
                     ▼                             ▼
              ┌──────────────┐             ┌──────────────┐
              │     AZ-1      │             │     AZ-2      │
              │ Public Subnet │             │ Public Subnet │
              │ 10.0.1.0/24  │             │ 10.0.2.0/24  │
              │              │             │              │
              │ Auto Scaling │             │ Auto Scaling │
              │ Group        │             │ Group        │
              │              │             │              │
              │ EC2 + Docker │             │ EC2 + Docker │
              └──────┬───────┘             └──────┬───────┘
                     │                            │
                     └────────────┬───────────────┘
                                  │
                                  ▼
                     ┌─────────────────────────┐
                     │     PRIVATE SUBNETS     │
                     │                         │
                     │      Amazon RDS          │
                     │        MySQL            │
                     │                         │
                     │      employees DB       │
                     └─────────────────────────┘
```

---

# 🔄 Application Request Flow

```text
User
 │
 ▼
Application Load Balancer :80
 │
 ▼
Target Group
 │
 ▼
EC2 Instance :80
 │
 ▼
Host Nginx
 │
 ▼
Frontend Docker Container :81
 │
 ▼
Backend Docker Container :5000
 │
 ▼
Amazon RDS MySQL :3306
```

---

# 🛠️ Technologies Used

| Technology                | Purpose                    |
| ------------------------- | -------------------------- |
| AWS VPC                   | Network isolation          |
| Amazon EC2                | Application servers        |
| Amazon RDS                | Managed MySQL database     |
| Application Load Balancer | Traffic distribution       |
| Auto Scaling              | Application scalability    |
| Docker                    | Containerization           |
| Nginx                     | Web server / reverse proxy |
| Flask                     | Backend API                |
| MySQL                     | Relational database        |
| Ubuntu                    | EC2 operating system       |
| Git/GitHub                | Version control            |

---

# 📂 Project Structure

```text
My-application/
│
├── backend/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
│
├── frontend/
│   ├── index.html
│   ├── script.js
│   ├── styles.css
│   ├── default.conf
│   └── Dockerfile
│
├── Database/
│   └── database-creation.txt
│
└── README.md
```

---

# ☁️ AWS Infrastructure Setup

## Step 1: Create the VPC

Go to:

```text
AWS Console
→ VPC
→ Your VPCs
→ Create VPC
```

Create the VPC:

```text
Name: vpc-for-2tier
IPv4 CIDR: 10.0.0.0/16
```

The VPC provides the isolated networking environment for the entire application.

---

# Step 2: Create Public Subnets

Create two public subnets across different Availability Zones.

### Public Subnet 1

```text
Name: public-subnet-1
Availability Zone: us-east-1a
CIDR: 10.0.1.0/24
```

### Public Subnet 2

```text
Name: public-subnet-2
Availability Zone: us-east-1b
CIDR: 10.0.2.0/24
```

These subnets will be used by the Application Load Balancer and application EC2 instances.

---

# Step 3: Create Database Subnets

Create two additional subnets for RDS.

### Database Subnet 1

```text
Name: db-subnet-1
Availability Zone: us-east-1a
CIDR: 10.0.3.0/24
```

### Database Subnet 2

```text
Name: db-subnet-2
Availability Zone: us-east-1b
CIDR: 10.0.4.0/24
```

These subnets are used for the database tier.

---

# Step 4: Create Internet Gateway

Go to:

```text
VPC
→ Internet Gateways
→ Create Internet Gateway
```

Name:

```text
My-IG
```

Attach the Internet Gateway to:

```text
vpc-for-2tier
```

---

# Step 5: Create Public Route Table

Go to:

```text
VPC
→ Route Tables
→ Create Route Table
```

Create:

```text
Name: PublicRouteTable
VPC: vpc-for-2tier
```

Add the following route:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

Associate this route table with:

```text
public-subnet-1
public-subnet-2
```

The final public route table should contain:

```text
Destination       Target
----------------------------------
10.0.0.0/16       local
0.0.0.0/0         igw-xxxxxxxx
```

---

# 🔐 Security Groups

Create three Security Groups:

```text
loadbalancer-sg
Application-server-SG
DBserver-SG
```

---

# Step 6: Load Balancer Security Group

Create:

```text
Name: loadbalancer-sg
```

### Inbound Rules

```text
Type: HTTP
Port: 80
Source: 0.0.0.0/0
```

### Outbound Rules

```text
All Traffic
Destination: 0.0.0.0/0
```

---

# Step 7: Application Server Security Group

Create:

```text
Name: Application-server-SG
```

### Inbound Rules

```text
HTTP
Port: 80
Source: loadbalancer-sg

SSH
Port: 22
Source: Your IP
```

If required for internal backend communication:

```text
Custom TCP
Port: 5000
Source: Appropriate internal security group
```

Avoid exposing application ports publicly unless required.

---

# Step 8: Database Security Group

Create:

```text
Name: DBserver-SG
```

### Inbound Rules

```text
MySQL/Aurora
Port: 3306
Source: Application-server-SG
```

Do **not** allow:

```text
0.0.0.0/0
```

for MySQL port `3306`.

The database should only accept traffic from the application tier.

---

# 🗄️ Amazon RDS MySQL

## Step 9: Create DB Subnet Group

Go to:

```text
RDS
→ Subnet Groups
→ Create DB Subnet Group
```

Configure:

```text
Name: two-tier-subnet-group
VPC: vpc-for-2tier
```

Add:

```text
db-subnet-1
db-subnet-2
```

---

# Step 10: Create RDS MySQL Database

Go to:

```text
RDS
→ Databases
→ Create Database
```

Select:

```text
Engine: MySQL
```

Example configuration:

```text
DB Identifier:
application-database

Master Username:
admin

Master Password:
YOUR_STRONG_PASSWORD
```

Choose an appropriate instance class for your AWS account/budget.

---

# Step 11: RDS Connectivity

Configure:

```text
VPC:
vpc-for-2tier

DB Subnet Group:
two-tier-subnet-group

Public Access:
No

VPC Security Group:
DBserver-SG
```

Wait until the RDS status becomes:

```text
Available
```

Then copy the RDS endpoint.

Example:

```text
application-database.xxxxxxxxx.us-east-1.rds.amazonaws.com
```

---

# 🖥️ EC2 Application Server

## Step 12: Launch EC2

Go to:

```text
EC2
→ Instances
→ Launch Instance
```

Example:

```text
Name:
myapplicationserver

AMI:
Ubuntu

Instance Type:
t2.micro / t3.micro

Key Pair:
aws-project.pem

VPC:
vpc-for-2tier

Subnet:
public-subnet-1

Security Group:
Application-server-SG
```

---

# Step 13: Connect to EC2

Connect using SSH:

```bash
ssh -i aws-project.pem ubuntu@<EC2_PUBLIC_IP>
```

Update the system:

```bash
sudo apt update
sudo apt upgrade -y
```

---

# 🐳 Docker Installation

## Step 14: Install Docker

Install Docker:

```bash
sudo apt install docker.io -y
```

Enable Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify:

```bash
docker --version
```

Test:

```bash
sudo docker run hello-world
```

---

# 📦 Clone the Repository

## Step 15: Clone Project

```bash
cd ~
git clone https://github.com/yashrpandit/My-application.git
```

Enter the project:

```bash
cd My-application
```

Check:

```bash
ls
```

Expected structure:

```text
backend
frontend
Database
```

---

# 🗄️ Database Configuration

## Step 16: Connect to RDS

Install MySQL client:

```bash
sudo apt update
sudo apt install mysql-client -y
```

Connect:

```bash
mysql -h <RDS_ENDPOINT> -u admin -p
```

Example:

```bash
mysql -h application-database.xxxxxxxxx.us-east-1.rds.amazonaws.com -u admin -p
```

---

# Step 17: Create Database

Inside MySQL:

```sql
CREATE DATABASE employees;
```

Use the database:

```sql
USE employees;
```

Create the table:

```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    position VARCHAR(100) NOT NULL,
    salary DECIMAL(10,2) NOT NULL
);
```

Verify:

```sql
SHOW DATABASES;
```

Check tables:

```sql
SHOW TABLES;
```

Check structure:

```sql
DESCRIBE employees;
```

---

# 🐍 Backend Configuration

## Step 18: Configure Flask Backend

Go to:

```bash
cd ~/My-application/backend
```

Update the database configuration in `app.py`.

Example:

```python
db_config = {
    "host": "YOUR_RDS_ENDPOINT",
    "user": "admin",
    "password": "YOUR_RDS_PASSWORD",
    "database": "employees",
    "port": 3306
}
```

Do not commit passwords or credentials to GitHub.

Use environment variables for production deployments.

---

# Step 19: Build Backend Docker Image

From the backend directory:

```bash
docker build -t backendapp .
```

Verify:

```bash
docker images
```

---

# Step 20: Run Backend Container

```bash
docker run -d \
  --name backendapp \
  -p 5000:5000 \
  backendapp
```

Check:

```bash
docker ps
```

You should see:

```text
0.0.0.0:5000->5000/tcp
```

Check backend logs:

```bash
docker logs backendapp
```

---

# Step 21: Test Backend

From EC2:

```bash
curl http://localhost:5000
```

If your API endpoint is available:

```bash
curl http://localhost:5000/<API_ENDPOINT>
```

The backend should be able to communicate with RDS.

---

# 🌐 Frontend Configuration

## Step 22: Configure Frontend

Go to:

```bash
cd ~/My-application/frontend
```

Check the frontend files:

```bash
ls
```

The frontend should communicate with the backend using the correct API endpoint.

Avoid hardcoding:

```text
localhost
```

in browser-side API requests.

For production, use the ALB hostname or route API traffic through Nginx/ALB.

---

# Step 23: Build Frontend Docker Image

```bash
docker build -t webapp .
```

Verify:

```bash
docker images
```

---

# Step 24: Run Frontend Container

Example:

```bash
docker run -d \
  --name frontend \
  -p 81:80 \
  webapp
```

Check:

```bash
docker ps
```

Expected:

```text
0.0.0.0:81->80/tcp
```

Test:

```bash
curl -I http://localhost:81
```

Expected:

```text
HTTP/1.1 200 OK
```

---

# 🌐 Nginx Configuration

## Step 25: Install Nginx

```bash
sudo apt install nginx -y
```

Start Nginx:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

Check:

```bash
sudo systemctl status nginx
```

---

# Step 26: Configure Nginx Reverse Proxy

Edit:

```bash
sudo nano /etc/nginx/sites-available/default
```

Configure:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:81;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Test:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

Test:

```bash
curl -I http://localhost:80
```

Expected:

```text
HTTP/1.1 200 OK
```

---

# ⚖️ Application Load Balancer

## Step 27: Create Target Group

Go to:

```text
EC2
→ Target Groups
→ Create Target Group
```

Configure:

```text
Target Type:
Instances

Name:
TG1

Protocol:
HTTP

Port:
80

VPC:
vpc-for-2tier
```

Health check:

```text
Protocol: HTTP
Port: Traffic Port
Path: /
```

---

# Step 28: Register EC2 Target

Register:

```text
myapplicationserver
```

Port:

```text
80
```

The target should eventually show:

```text
Healthy
```

---

# Step 29: Create Application Load Balancer

Go to:

```text
EC2
→ Load Balancers
→ Create Load Balancer
```

Select:

```text
Application Load Balancer
```

Configure:

```text
Name:
publicfacing-LB

Scheme:
Internet-facing

IP Address Type:
IPv4
```

Select:

```text
VPC:
vpc-for-2tier
```

Select both public subnets:

```text
public-subnet-1
public-subnet-2
```

Security Group:

```text
loadbalancer-sg
```

Listener:

```text
HTTP :80
```

Forward traffic to:

```text
TG1
```

---

# Step 30: Test ALB

Copy the ALB DNS name.

Example:

```text
publicfacing-lb-xxxxxxxx.us-east-1.elb.amazonaws.com
```

Test:

```bash
curl -I http://<ALB_DNS_NAME>
```

Expected:

```text
HTTP/1.1 200 OK
```

Open in browser:

```text
http://<ALB_DNS_NAME>
```

---

# 📈 Auto Scaling

## Step 31: Create AMI

Once the application is completely working on the EC2 instance:

Go to:

```text
EC2
→ Instances
→ Select application server
→ Actions
→ Image and templates
→ Create image
```

Example:

```text
Name:
imagefor2tier
```

The AMI should contain the required application environment.

---

# Step 32: Create Launch Template

Go to:

```text
EC2
→ Launch Templates
→ Create launch template
```

Configure:

```text
Name:
two-tier-launch-template
```

Select:

```text
AMI:
imagefor2tier

Instance Type:
t3.micro

Key Pair:
aws-project.pem

Security Group:
Application-server-SG
```

Do not hardcode a specific subnet in the launch template if the Auto Scaling Group will determine the subnet placement.

---

# Step 33: Create Auto Scaling Group

Go to:

```text
EC2
→ Auto Scaling Groups
→ Create Auto Scaling Group
```

Select:

```text
Launch Template:
two-tier-launch-template
```

Choose:

```text
VPC:
vpc-for-2tier
```

Select:

```text
public-subnet-1
public-subnet-2
```

Example capacity:

```text
Desired Capacity:
2

Minimum Capacity:
1

Maximum Capacity:
3
```

---

# Step 34: Attach Target Group

Attach:

```text
TG1
```

Enable:

```text
Application Load Balancer
```

The Auto Scaling Group will automatically register new EC2 instances with the target group.

---

# 🔄 Auto Scaling Flow

```text
                    Application Load Balancer
                              │
                     ┌────────┴────────┐
                     │                 │
                     ▼                 ▼
                  EC2 #1            EC2 #2
                     │                 │
                     └────────┬────────┘
                              │
                     Auto Scaling Group
                              │
                       Scale Out / In
```

---

# 🧪 Testing the Complete Application

## Test Frontend

```bash
curl -I http://localhost:80
```

---

## Test Frontend Container

```bash
curl -I http://localhost:81
```

---

## Test Backend

```bash
curl -I http://localhost:5000
```

---

## Test ALB

```bash
curl -I http://<ALB_DNS_NAME>
```

Expected:

```text
HTTP/1.1 200 OK
```

---

## Test Database

```bash
mysql \
-h <RDS_ENDPOINT> \
-u admin \
-p
```

Then:

```sql
USE employees;
SELECT * FROM employees;
```

---

# 🐳 Useful Docker Commands

List containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

View logs:

```bash
docker logs <container_name>
```

Follow logs:

```bash
docker logs -f <container_name>
```

Stop container:

```bash
docker stop <container_name>
```

Remove container:

```bash
docker rm <container_name>
```

Remove image:

```bash
docker rmi <image_name>
```

Build image:

```bash
docker build -t <image_name> .
```

---

# 🧰 Useful Nginx Commands

Check status:

```bash
sudo systemctl status nginx
```

Test configuration:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

View active configuration:

```bash
sudo nginx -T
```

Check listening ports:

```bash
sudo ss -lntp
```

---

# 🛠️ Troubleshooting

## Problem 1: RDS DNS `NXDOMAIN`

Error:

```text
ERROR 2005 (HY000): Unknown MySQL server host
```

Check:

```bash
nslookup <RDS_ENDPOINT>
```

Verify:

* Correct RDS endpoint
* VPC DNS resolution
* VPC DNS hostnames
* Correct region
* RDS instance status is `Available`

---

# Problem 2: MySQL Access Denied

Error:

```text
ERROR 1045 (28000): Access denied
```

Check:

* RDS username
* RDS password
* Database credentials
* Master username
* Application configuration

Test manually:

```bash
mysql -h <RDS_ENDPOINT> -u admin -p
```

---

# Problem 3: ALB Connection Timeout

Error:

```text
ERR_CONNECTION_TIMED_OUT
```

Check:

```text
ALB status
Listener
Security Group
Target Group
Target Health
Subnets
Route Tables
Internet Gateway
```

ALB Security Group should allow:

```text
HTTP :80
Source: 0.0.0.0/0
```

---

# Problem 4: Target Unhealthy

Check:

```text
Target Group
→ Targets
```

Verify:

```text
Port
Health Check Path
Security Group
Application Status
```

Test directly from EC2:

```bash
curl -I http://localhost:80
```

---

# Problem 5: Frontend Works but Buttons Don't

Open browser Developer Tools:

```text
F12
→ Console
→ Network
→ Fetch/XHR
```

Check whether API requests are going to:

```text
localhost:5000
```

or an incorrect EC2 IP.

Browser-side JavaScript should not rely on `localhost` for a remote backend.

Use an appropriate API endpoint routed through the application architecture.

---

# Problem 6: ALB Shows Wrong Nginx Page

Check what is listening:

```bash
sudo ss -lntp | grep -E ':80|:81|:5000'
```

Example:

```text
:80    → Host Nginx
:81    → Frontend Docker
:5000  → Backend Docker
```

If the host Nginx serves the default page, configure it to proxy to the frontend container:

```nginx
location / {
    proxy_pass http://127.0.0.1:81;
}
```

Then:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# Problem 7: ALB Has Multiple Targets Serving Different Versions

If one target returns different content from another:

```text
Target 1 → New Application
Target 2 → Old Application
```

Check:

* AMI
* Launch Template
* Auto Scaling Group
* Docker images
* Nginx configuration

All Auto Scaling instances should use the same application version.

---

# 🔒 Security Best Practices

Do not expose the RDS database publicly.

```text
RDS Public Access:
No
```

Allow MySQL only from:

```text
Application-server-SG
```

Do not expose port `3306` to:

```text
0.0.0.0/0
```

Do not expose frontend container ports publicly if the ALB is the intended entry point.

SSH should preferably be restricted to your IP:

```text
Port 22
Source: YOUR_IP/32
```

Never commit:

```text
Passwords
AWS credentials
Private keys
.env files
```

to GitHub.

---

# 📊 Final Architecture

```text
                         INTERNET
                            │
                            ▼
                ┌───────────────────────┐
                │ Application Load      │
                │ Balancer              │
                │ HTTP :80              │
                └───────────┬───────────┘
                            │
               ┌────────────┴────────────┐
               │                         │
               ▼                         ▼
        ┌──────────────┐          ┌──────────────┐
        │     AZ-1     │          │     AZ-2     │
        │ Public       │          │ Public       │
        │ Subnet       │          │ Subnet       │
        │              │          │              │
        │ EC2          │          │ EC2          │
        │ Docker       │          │ Docker       │
        │ Nginx        │          │ Nginx        │
        └──────┬───────┘          └──────┬───────┘
               │                         │
               └────────────┬────────────┘
                            │
                            ▼
                  ┌──────────────────┐
                  │  PRIVATE SUBNETS │
                  │                  │
                  │ Amazon RDS       │
                  │ MySQL            │
                  │                  │
                  │ employees DB     │
                  └──────────────────┘

                    AUTO SCALING
                         │
                         ▼
              EC2 instances scale
              based on application
                     demand
```

---

# 🎯 Key Learning Outcomes

Through this project, I gained hands-on experience with:

* AWS VPC architecture
* Public and private subnet design
* Internet Gateway
* Route Tables
* Security Groups
* Amazon EC2
* Amazon RDS MySQL
* Application Load Balancer
* Target Groups
* Health Checks
* Auto Scaling Groups
* Launch Templates
* AMIs
* Docker containerization
* Nginx reverse proxy
* Flask backend
* MySQL connectivity
* DNS troubleshooting
* Network troubleshooting
* Application debugging
* High availability concepts

---

# 🚀 Future Improvements

Possible improvements for this project:

* Add HTTPS using AWS Certificate Manager
* Configure Route 53 custom domain
* Add CloudWatch monitoring
* Add centralized logging
* Add AWS Secrets Manager
* Implement CI/CD using Jenkins or GitHub Actions
* Add container image scanning using Trivy
* Deploy using ECS/EKS
* Add WAF protection
* Add private EC2 subnets with NAT Gateway
* Implement blue-green or rolling deployments

---

# 📚 What I Learned

The biggest lesson from this project was that **deploying an application is only one part of DevOps**.

Real-world cloud deployments require continuous troubleshooting.

During this project, I worked through issues involving:

```text
DNS Resolution
MySQL Authentication
Security Groups
ALB Configuration
Target Health
Route Tables
Internet Gateway
Docker Ports
Nginx Configuration
Frontend API Connectivity
Auto Scaling Targets
```

Instead of changing random configurations, I learned to troubleshoot the architecture layer by layer:

```text
Client
  ↓
ALB
  ↓
Target Group
  ↓
EC2
  ↓
Nginx
  ↓
Docker
  ↓
Backend
  ↓
RDS
```

This approach made it easier to identify exactly where a failure was occurring.

---

# 🔗 Repository

GitHub:

https://github.com/yashrpandit/My-application

---

# 👨‍💻 Author

**Mohammad Kasim**

DevOps & Cloud Engineering Learner

---

# ⭐ If You Found This Project Useful

Give the repository a ⭐ on GitHub and feel free to explore, fork, or improve the project.

**Build → Deploy → Break → Debug → Learn → Repeat 🚀**
