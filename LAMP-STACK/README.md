# LAMP Stack Implementation in AWS

## Introduction

The **LAMP** stack is a widely-used open-source web development platform that is reliable and flexible. It consists of four key components:
- **Linux**: Operating System
- **Apache**: Web Server
- **MySQL**: Database Management System
- **PHP**: Server-Side Programming Language

This guide details the steps to install, configure, and deploy a web application using the LAMP stack on an AWS EC2 instance.

## Prerequisites

1. An **EC2 instance** of type `t2.micro` with **Ubuntu 24.04 LTS (HVM)** launched in the **us-east-1 region**.
2. **SSH Key Pair** named `my-ec2-key` to access the instance on port `22`.
3. Configured **security group** with the following inbound rules:
   - Port `80` (HTTP) open to all.
   - Port `443` (HTTPS) open to all.
   - Port `22` (SSH) open to all.
4. **Default VPC** and **Subnet** used for networking.

## Step 0: Connect to EC2

1. Use the SSH key pair to connect to your EC2 instance:
    ```bash
    chmod 400 my-ec2-key.pem
    ssh -i "my-ec2-key.pem" ubuntu@<your-public-ip>
    ```

## Step 1: Install Apache and Update the Firewall

1. **Update and upgrade** the package manager:
    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```

2. **Install Apache**:
    ```bash
    sudo apt install apache2 -y
    ```

3. **Enable and check** Apache service:
    ```bash
    sudo systemctl enable apache2
    sudo systemctl status apache2
    ```

4. Test Apache locally:
    ```bash
    curl http://localhost:80
    ```

5. Test Apache with the public IP:
    ```bash
    http://<your-public-ip>
    ```

## Step 2: Install MySQL

1. **Install MySQL**:
    ```bash
    sudo apt install mysql-server
    ```

2. **Enable and check** MySQL service:
    ```bash
    sudo systemctl enable --now mysql
    sudo systemctl status mysql
    ```

3. **Log in** to the MySQL console:
    ```bash
    sudo mysql -p
    ```

4. **Set a root password**:
    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
    ```

## Step 3: Install PHP

1. **Install PHP and modules**:
    ```bash
    sudo apt install php libapache2-mod-php php-mysql
    ```

2. **Check PHP version**:
    ```bash
    php -v
    ```

## Step 4: Create a Virtual Host for the Website

1. **Create the project directory**:
    ```bash
    sudo mkdir /var/www/projectlamp
    sudo chown -R $USER:$USER /var/www/projectlamp
    ```

2. **Create a new Apache configuration file**:
    ```bash
    sudo vim /etc/apache2/sites-available/projectlamp.conf
    ```

3. Paste the following into the file:
    ```apache
    <VirtualHost *:80>
        ServerName projectlamp
        ServerAlias www.projectlamp
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/projectlamp
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```

4. **Enable the new virtual host**:
    ```bash
    sudo a2ensite projectlamp
    sudo a2dissite 000-default
    ```

5. **Check Apache configuration**:
    ```bash
    sudo apache2ctl configtest
    ```

6. **Reload Apache**:
    ```bash
    sudo systemctl reload apache2
    ```

## Step 5: Enable PHP for the Website

1. **Update the DirectoryIndex settings** in Apache to prioritize `index.php`:
    ```bash
    sudo vim /etc/apache2/mods-enabled/dir.conf
    ```
    Change this:
    ```apache
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
    ```
    To this:
    ```apache
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
    ```

2. **Reload Apache**:
    ```bash
    sudo systemctl reload apache2
    ```

3. **Create a PHP test file**:
    ```bash
    vim /var/www/projectlamp/index.php
    ```
    Add the following code:
    ```php
    <?php
    phpinfo();
    ```

4. **Access the PHP page**:
    Visit the public IP:
    ```bash
    http://<your-public-ip>
    ```

5. **Remove the test file** for security:
    ```bash
    sudo rm /var/www/projectlamp/index.php
    ```

## Conclusion

By following the steps outlined above, you have successfully set up a LAMP stack on AWS EC2. This provides a solid environment for deploying and managing dynamic web applications.

