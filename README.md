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
* Nginx
* Java
* Apache Tomcat
* Amazone RDS
* MySQL Connector/J
* Amazon VPC

---

# 🌐 VPC & Networking Setup

## 1. Create VPC

```
VPC CIDR: 10.0.0.0/16
```

---

## 2. Create Subnets

### 🔹 Public Subnet

```
CIDR: 10.0.0.0/20
```

* Used for Reverse Proxy EC2
* Internet accessible

### 🔹 Private Subnet

```
CIDR: 10.0.16.0/20
```

* Used for Backend EC2 & RDS
* No direct internet access
* Use **NAT** Gateway
---

## 3. Internet Gateway (IGW)

* Create IGW
* Attach to VPC

---

## 4. Route Tables

### Public Route Table

```
Destination: 0.0.0.0/0 → Internet Gateway
```

Associate with Public Subnet

---

### Private Route Table

```
No direct internet route
Internet access through NAT gateway
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

# 🖥️ EC2 Setup & Application Deployment

## Proxy Server (Public Subnet)

* Install Nginx
  
  ```bash
  sudo yum update
  sudo yum install nginx -y
  sudo systemctl start nginx
  sudo systemctl enable nginx
  ```
  
* Make configuration
  
  ```bash
  cd /etc/nginx/
  sudo vim nginx.conf
  ```
* Add proxy to location block
> location / {
> proxy_pass http://<Backend_private_IP>:8080/student/;
>}
---

## Backend Server (Private Subnet)

* Install Java & Apache Tomcat
  
  ```bash
  sudo yum update
  sudo yum install java -y
  sudo curl -O https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.116/bin/apache-tomcat-9.0.116.tar.gz
  ```
* Extract the apache-tomcat.tar.gz file in **/opt** directory
  
  ```bash
  sudo tar -xvzf apache-tomcat-9.0.116.tar.gz -C /opt
  cd /opt
  ```
* Rename the unzip file
  
  ```bash
  sudo mv apache-tomcat-9.0.116.tar.gz apache
  ```
* Deploy **student.war** file
  
  ```bash
  sudo -i
  cd /opt/apache/webapps/
  curl -o https://s3-us-west-2.amazonaws.com/studentapi-cit/student.war
  ```
* Restart Tomcat
  
  ```bash
  cd /opt/apache/bin/
  ./catalina.sh stop
  ./catalina.sh start
  ```
* Now to check Tomcat work or not Hit the proxy-public-IP  
* Runs on port 8080

---


---

# 🗄️ Database Setup (RDS)

* Goto RDS and Create Database
  1. Choose - `custom vpc`
  2. Security Group - `3306`
  3. User - `admin`
  4. Password - `<PASSWORD>`

### Access RDS or Connect RDS

* Install mariadb on app-server

```bash
sudo yum install mariadb105-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb
```
* Connect to RDS

```bash
sudo mysql -u admin -p -h <RDS_ENDPOINT>
```

* Inside Mysql create Database and Table

```sql
CREATE DATABASE studentapp;
USE studentapp;

CREATE TABLE students (
    student_id INT AUTO_INCREMENT PRIMARY KEY,
    student_name VARCHAR(100),
    student_addr VARCHAR(100),
    student_age VARCHAR(3),
    student_qual VARCHAR(20),
    student_percent VARCHAR(10),
    student_year_passed VARCHAR(10)
);
```

---

# 🔌 Install MySQL Connector (On app-server)

```bash
cd /opt/apache/lib
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/mysql-connector.jar
```

* For access Application Edit configuration file of Apache-Tomcat

```bash
cd /opt/apache/conf/
vim context.xml
```

* Add this inside context block

```bash
<Resource name="jdbc/TestDB" auth="Container"
          type="javax.sql.DataSource"
          maxTotal="500" maxIdle="30" maxWaitMillis="1000"
          username="<USER>" password="<YOUR_PASSWORD>"
          driverClassName="com.mysql.jdbc.Driver"
          url="jdbc:mysql://<RDS-ENDPOINT>:3306/studentapp?useUnicode=yes&amp;characterEncoding=utf8"/>
```

* Restart the Apache-tomcat

```bash
cd /opt/apache/bin
./catalina.sh stop
./catalina.sh start
 ```
 
---



---

# 🔐 Security Implementation

* Backend EC2 is in **private subnet**
* No public IP assigned
* Only Proxy EC2 can access backend
* RDS only accessible from backend

---

# 🌐 Access Application

```
http://<Proxy-Public-IP>
```

---


### 2. Database Verification

```sql
SELECT * FROM students;
```

# 🔄 Traffic Flow

1. User → Proxy (Public Subnet)
2. Proxy → Backend (Private Subnet)
3. Backend → RDS (Private Subnet)
4. Response → Proxy → User

---

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
