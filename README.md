# LAMP-STACK
**L=Linux
A=Apache
M=MySQL
P=PHP**
# WordPress Hosting on AWS (EC2 + RDS with LAMP Stack)
This project demonstrates hosting a WordPress website on **Amazon EC2** with **Amazon RDS (MySQL)** using the **LAMP stack (Linux, Apache, MySQL, PHP)**.
**1.Architecture**
- Amazon Linux 2023 EC2 instance
- Apache + PHP + WordPress
- Amazon RDS (MySQL, private access)
- Auto Scaling Group + Load Balancer for high availability
- Blue/Green deployment strategy
**2. Setup Instructions**
1. Launch EC2 (Amazon Linux 2023).
2. Install MySQL client:
dnf install -y mariadb105
**3.Connect to RDS and create DB + user:**
CREATE DATABASE wordpress;
CREATE USER 'wpuser' IDENTIFIED BY 'root123456';
GRANT ALL PRIVILEGES ON wordpress.* TO wpuser;
FLUSH PRIVILEGES;
**4.Install Apache + PHP:**
sudo yum install -y httpd php8.4 php-mysqlnd
sudo service httpd start
**5.Download and configure WordPress:**
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
Update wp-config.php with RDS endpoint, DB name, user, and password.
**6.Copy files to /var/www/html and restart Apache:**
sudo cp -r wordpress/* /var/www/html/
sudo service httpd restart
**8.Access via browser:**
1.http://<EC2-Public-IP> → Website
2.http://<EC2-Public-IP>/wp-admin → Admin panel
**9.Blue/Green Deployment:**
1.Create AMI of WordPress EC2.
2.Launch new EC2 from AMI.
3.Configure Target Group + ALB.
4.Use Auto Scaling Group with Launch Template.



  
