# LAMP-STACK

## LAMP Stack

LAMP stands for:

* **L** → Linux
* **A** → Apache
* **M** → MySQL
* **P** → PHP

LAMP is a popular technology stack used to host dynamic web applications such as WordPress.

---

# WordPress Hosting on AWS

This project demonstrates how to host a **WordPress website on AWS EC2** using the **LAMP stack**, with **Amazon RDS MySQL** as the database.

## Architecture

```text
                 User
                   |
                   v
            Load Balancer
                   |
                   v
              EC2 Instance
                   |
        +----------+----------+
        |                     |
        v                     v
     Apache                 PHP
        |                     |
        +----------+----------+
                   |
                   v
             WordPress
                   |
                   v
              Amazon RDS
                MySQL
```

### AWS Components

* Amazon EC2
* Amazon RDS MySQL
* Application Load Balancer
* Auto Scaling Group
* Security Groups
* Amazon Linux 2023
* Apache
* PHP
* WordPress

---

# 1. Launch EC2 Instance

Launch an EC2 instance with:

* **OS:** Amazon Linux 2023
* **Instance Type:** Suitable instance for the lab
* **Storage:** As required
* **Security Group:** Allow HTTP (80) and SSH (22)

Connect to the EC2 instance using SSH.

```bash
ssh -i key.pem ec2-user@<EC2-PUBLIC-IP>
```

---

# 2. Install MySQL Client

Install the MariaDB/MySQL client on the EC2 server.

```bash
sudo dnf install -y mariadb105
```

The client is used to connect from EC2 to the RDS MySQL database.

---

# 3. Create Amazon RDS MySQL

Create an Amazon RDS database with:

* **Engine:** MySQL
* **Database Name:** wordpress
* **Username:** wpuser
* **Password:** <your-password>
* **Connectivity:** Private access recommended

Make sure the RDS Security Group allows MySQL traffic on:

```text
Port: 3306
Source: EC2 Security Group
```

---

# 4. Connect EC2 to RDS

From the EC2 instance:

```bash
mysql -h <RDS-ENDPOINT> -u admin -p
```

Enter the RDS password.

After connecting, create the WordPress database:

```sql
CREATE DATABASE wordpress;
```

Create a database user:

```sql
CREATE USER 'wpuser' IDENTIFIED BY '<your-password>';
```

Grant permissions:

```sql
GRANT ALL PRIVILEGES ON wordpress.* TO wpuser;
```

Apply the changes:

```sql
FLUSH PRIVILEGES;
```

Verify:

```sql
SHOW DATABASES;
```

---

# 5. Install Apache and PHP

Install Apache:

```bash
sudo yum install -y httpd
```

Install PHP and MySQL PHP extension:

```bash
sudo yum install -y php8.4 php-mysqlnd
```

Start Apache:

```bash
sudo systemctl start httpd
```

Enable Apache at boot:

```bash
sudo systemctl enable httpd
```

Check Apache status:

```bash
sudo systemctl status httpd
```

---

# 6. Test Apache

Create a simple HTML file:

```bash
echo "Hello from Apache" | sudo tee /var/www/html/index.html
```

Open in a browser:

```text
http://<EC2-PUBLIC-IP>
```

If the page opens, Apache is working.

---

# 7. Download WordPress

Download the latest WordPress package:

```bash
wget https://wordpress.org/latest.tar.gz
```

Extract the package:

```bash
tar -xzf latest.tar.gz
```

This creates a directory:

```text
wordpress/
```

---

# 8. Configure WordPress

Move into the WordPress directory:

```bash
cd wordpress
```

Create the WordPress configuration file:

```bash
cp wp-config-sample.php wp-config.php
```

Edit the configuration:

```bash
vi wp-config.php
```

Configure the database values:

```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', '<your-password>' );
define( 'DB_HOST', '<RDS-ENDPOINT>' );
```

### Database Flow

```text
WordPress
    |
    v
PHP
    |
    v
MySQL Client
    |
    v
Amazon RDS MySQL
```

---

# 9. Copy WordPress Files

Copy the WordPress files to Apache's document root:

```bash
sudo cp -r wordpress/* /var/www/html/
```

Set appropriate ownership:

```bash
sudo chown -R apache:apache /var/www/html
```

Restart Apache:

```bash
sudo systemctl restart httpd
```

---

# 10. Access WordPress

Open the following URL:

```text
http://<EC2-PUBLIC-IP>
```

WordPress installation page should appear.

For the administrator panel:

```text
http://<EC2-PUBLIC-IP>/wp-admin
```

Complete the WordPress installation.

---

# 11. Security Groups

### EC2 Security Group

Allow:

```text
SSH   → 22
HTTP  → 80
```

HTTPS can also be allowed when SSL is configured:

```text
HTTPS → 443
```

### RDS Security Group

Allow:

```text
MySQL/Aurora → 3306
Source → EC2 Security Group
```

It is preferable to allow database access from the EC2 security group instead of opening port 3306 to the internet.

---

# 12. Basic Architecture

```text
                  Internet
                     |
                     v
              Load Balancer
                     |
          +----------+----------+
          |                     |
          v                     v
       EC2-01                 EC2-02
          |                     |
       Apache                 Apache
          |                     |
        PHP                  PHP
          |                     |
       WordPress             WordPress
          |                     |
          +----------+----------+
                     |
                     v
              Amazon RDS
                 MySQL
```

---

# 13. Auto Scaling

An Auto Scaling Group can be used to automatically maintain the required number of EC2 instances.

Example:

```text
Minimum Capacity = 2
Desired Capacity = 2
Maximum Capacity = 4
```

When traffic increases, Auto Scaling can launch additional EC2 instances.

When traffic decreases, instances can be reduced according to the configured scaling policy.

---

# 14. Load Balancer

An Application Load Balancer distributes incoming requests across multiple WordPress EC2 instances.

```text
                Users
                  |
                  v
           Application LB
             /          \
            /            \
           v              v
        EC2-01          EC2-02
           \              /
            \            /
             v          v
              RDS MySQL
```

### Benefits

* Distributes traffic
* Improves availability
* Supports multiple EC2 instances
* Works with Auto Scaling
* Helps avoid dependence on a single web server

---

# 15. Blue/Green Deployment

Blue/Green deployment uses two environments.

```text
             Load Balancer
                  |
          +-------+-------+
          |               |
          v               v
       BLUE             GREEN
      Current            New
      Version           Version
```

### Blue Environment

The current production version.

### Green Environment

The new version being tested.

After validating the Green environment, traffic can be switched to it.

---

# 16. Blue/Green Deployment Steps

### Step 1

Create an AMI from the existing WordPress EC2 instance.

### Step 2

Launch a new EC2 instance from the AMI.

### Step 3

Configure the new instance.

### Step 4

Create/configure a Target Group.

### Step 5

Attach the Target Group to the Application Load Balancer.

### Step 6

Configure an Auto Scaling Group using a Launch Template.

### Step 7

Test the new WordPress environment.

### Step 8

Switch traffic to the new environment.

---

# 17. Important Concepts Learned

Through this project, I learned:

* LAMP architecture
* Linux server administration
* Apache web server
* PHP
* WordPress deployment
* Amazon EC2
* Amazon RDS MySQL
* EC2-to-RDS connectivity
* MySQL database configuration
* Security Groups
* Application Load Balancer
* Auto Scaling Groups
* AMI creation
* Launch Templates
* Blue/Green deployment

---

# 18. Final Architecture

```text
                         Users
                           |
                           v
                    Application LB
                           |
                 +---------+---------+
                 |                   |
                 v                   v
              EC2-01             EC2-02
                 |                   |
              Apache              Apache
                 |                   |
                PHP                 PHP
                 |                   |
             WordPress           WordPress
                 |                   |
                 +---------+---------+
                           |
                           v
                    Amazon RDS MySQL
                           |
                           v
                     WordPress DB
```

---

# Conclusion

This project helped me understand how a WordPress application can be deployed on AWS using a **LAMP stack**.

The application uses:

```text
Linux
  ↓
Apache
  ↓
PHP
  ↓
WordPress
  ↓
Amazon RDS MySQL
```

For higher availability and scalability, the architecture can be extended with:

```text
Application Load Balancer
        ↓
Auto Scaling Group
        ↓
Multiple EC2 Instances
        ↓
Amazon RDS MySQL
```

This project gave me practical experience with AWS infrastructure, Linux administration, web-server configuration, database connectivity, and basic high-availability deployment concepts.
