![Alt text](/2._Host_a_WordPress_Website_on_AWS.png)

---

# **Deploy a WordPress Website on AWS**  

This project demonstrates how to deploy a **WordPress website** on **AWS**, utilizing **EC2**, **EFS**, **RDS**, and other AWS services for a **scalable, secure, and high-availability** setup.

## **Architecture Overview**  

The WordPress website is hosted on **EC2 instances** and leverages AWS services to enhance security, scalability, and reliability. Below are the key components:

### **Infrastructure Setup**  
- Configured a **Virtual Private Cloud (VPC)** with **public and private subnets** across **two Availability Zones**.  
- Deployed an **Internet Gateway** for internet connectivity.  
- Created **Security Groups** to manage inbound and outbound traffic.  
- Used **Public Subnets** for NAT Gateway and Application Load Balancer.  
- Positioned **web servers (EC2 instances) in Private Subnets** for security.  
- Enabled private instances to access the Internet via a **NAT Gateway**.  

### **Compute & Storage**  
- **Auto Scaling Group**: Ensures website availability and scalability.  
- **Amazon EFS (Elastic File System)**: Shared storage for WordPress files.  
- **Amazon RDS (Relational Database Service)**: Managed MySQL database for WordPress.  

### **Security & Monitoring**  
- **AWS Certificate Manager**: Secures communication with SSL/TLS.  
- **AWS Simple Notification Service (SNS)**: Sends alerts for Auto Scaling activities.  

### **Domain Management**  
- Registered a **domain name** and configured a **DNS record using Route 53**.  

---

## **Deployment Instructions**  

### **Step 1: Setup EC2 Instance & Install Dependencies**  
Run the following script to install required packages and configure the environment:

```bash
#!/bin/bash

# Switch to root user
sudo su

# Update installed packages
sudo yum update -y

# Create an HTML directory
sudo mkdir -p /var/www/html

# Set environment variable for EFS
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount the EFS to the HTML directory
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache and enable it to start on boot
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and required extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server

# Start and enable MySQL
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html

# Download and setup WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Configure WordPress
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo vi /var/www/html/wp-config.php

# Restart Apache server
sudo service httpd restart
```

---

### **Step 2: Configure Auto Scaling Group**  
The following script is used in the **Launch Template** for the **Auto Scaling Group**:

```bash
#!/bin/bash

# Update packages
sudo yum update -y

# Install Apache and enable it to start on boot
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and required extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server

# Start and enable MySQL
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set environment variable for EFS
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com

# Mount EFS to HTML directory
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart Apache
sudo service httpd restart
```

---

## **Key Features**  
✅ Highly Available & Scalable WordPress Hosting  
✅ Secure Deployment with **AWS Certificate Manager**  
✅ Automatic Scaling via **Auto Scaling Group**  
✅ **EFS for shared file system**, **RDS for database storage**  
✅ **Domain Management using Route 53**  
✅ **Monitoring & Alerts via SNS**  

---

This project follows **AWS best practices** for **hosting WordPress**, leveraging **EC2, Auto Scaling, EFS, RDS, and Route 53** for a **scalable, secure, and highly available** deployment.
