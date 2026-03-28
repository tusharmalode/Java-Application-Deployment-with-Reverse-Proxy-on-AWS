# 🚀 Java Application Deployment with Reverse Proxy on AWS 

---

## 📌 Project Overview

This project demonstrates secure deployment of a Java-based Student Registration Web Application on AWS using a **reverse proxy architecture with VPC networking**. The backend server is deployed in a **private subnet**, ensuring it is not directly accessible from the internet.

---

## 🏗️ Architecture Overview

```
                    Internet
                        │
                        ▼
            Reverse Proxy (Public EC2)
                [Public Subnet]
                        │
                        ▼
            Backend Server (Tomcat)
                [Private Subnet]
                        │
                        ▼
            Amazon RDS (MySQL DB)
                [Private Subnet]
```

---

## 🎯 Objective

* Deploy Java WAR application on Apache Tomcat
* Use Amazon RDS for database
* Configure Nginx reverse proxy
* Implement secure networking using VPC
* Restrict backend access via private subnet

---

## 🛠️ Technologies Used

* AWS EC2
* AWS RDS (MySQL)
* Apache Tomcat
* Nginx
* MySQL Connector/J
* Amazon VPC

---

# 🌐 VPC & Networking Setup

## 1. Create VPC

```bash
VPC CIDR: 10.0.0.0/16
```

---

## 2. Create Subnets

### 🔹 Public Subnet

```bash
CIDR: 10.0.1.0/24
```

* Used for Reverse Proxy EC2
* Internet accessible

### 🔹 Private Subnet

```bash
CIDR: 10.0.2.0/24
```

* Used for Backend EC2 & RDS
* No direct internet access

---

## 3. Internet Gateway (IGW)

* Create IGW
* Attach to VPC

---

## 4. Route Tables

### Public Route Table

```bash
Destination: 0.0.0.0/0 → Internet Gateway
```

Associate with Public Subnet

---

### Private Route Table

```bash
No direct internet route
```

Associate with Private Subnet

---

## 5. Security Groups

### 🔹 Proxy EC2 SG

* Allow:

  * HTTP (80) → 0.0.0.0/0
  * SSH (22) → Your IP

---

### 🔹 Backend EC2 SG

* Allow:

  * Port 8080 → Proxy SG only
  * SSH (22) → Your IP

---

### 🔹 RDS SG

* Allow:

  * Port 3306 → Backend SG only

---

# 🖥️ EC2 Setup

## Backend Server (Private Subnet)

* Install Java & Apache Tomcat
* Deploy `student.war`
* Runs on port 8080

---

## Reverse Proxy Server (Public Subnet)

* Install Nginx
* Forward traffic to backend private IP

---

# ☕ Application Deployment

### Install Java

```bash
sudo yum install java-11-amazon-corretto -y
```

### Install Tomcat

```bash
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.85/bin/apache-tomcat-9.0.85.tar.gz
tar -xvzf apache-tomcat-9.0.85.tar.gz
mv apache-tomcat-9.0.85 tomcat
```

### Start Tomcat

```bash
cd tomcat/bin
chmod +x *.sh
./startup.sh
```

### Deploy WAR

```bash
cp student.war ~/tomcat/webapps/
```

---

# 🗄️ Database Setup (RDS)

### Create DB & Table

```sql
CREATE DATABASE studentdb;
USE studentdb;

CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    course VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

# 🔌 MySQL Connector

```bash
wget mysql-connector-java-8.0.xx.jar
mv mysql-connector-java-8.0.xx.jar ~/tomcat/lib/
```

Restart Tomcat after adding.

---

# 🔁 Reverse Proxy Configuration

### Install Nginx

```bash
sudo yum install nginx -y
```

### Configure

```bash
sudo vi /etc/nginx/nginx.conf
```

```nginx
location /student/ {
    proxy_pass http://<BACKEND_PRIVATE_IP>:8080/student/;
}
```

### Restart

```bash
sudo systemctl restart nginx
```

---

# 🔐 Security Implementation

* Backend EC2 is in **private subnet**
* No public IP assigned
* Only Proxy EC2 can access backend
* RDS only accessible from backend

---

# 🌐 Access Application

```
http://<Proxy-Public-IP>/student
```

---

# 🔄 Traffic Flow

1. User → Proxy (Public Subnet)
2. Proxy → Backend (Private Subnet)
3. Backend → RDS (Private Subnet)
4. Response → Proxy → User

---

# 📸 Deliverables

### 1. Application URL

```
http://<Proxy-Public-IP>/student
```

---

### 2. Database Verification

```sql
SELECT * FROM students;
```

---

# ⚠️ Challenges & Solutions

| Challenge                 | Solution                    |
| ------------------------- | --------------------------- |
| Private EC2 not reachable | Used Bastion/SSH from Proxy |
| DB connection failed      | Fixed SG rules              |
| Nginx 502 error           | Checked backend IP/port     |
| WAR not deploying         | Verified Tomcat logs        |

---

# ✅ Key Features

* Secure VPC architecture
* Public & Private subnet isolation
* Reverse proxy implementation
* Backend protection
* Scalable cloud deployment

---

# 📚 Conclusion

This project demonstrates how to securely deploy a Java web application using AWS VPC, ensuring backend isolation and controlled access through a reverse proxy.

---

## 👨‍💻 Author

**Your Name**
