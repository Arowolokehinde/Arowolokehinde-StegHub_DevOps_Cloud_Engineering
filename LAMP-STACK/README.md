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

![Screenshot 2024-09-14 111739](https://github.com/user-attachments/assets/55a9569b-3249-4fa7-9521-e2788bdfa32b)

![Screenshot 2024-09-14 111938](https://github.com/user-attachments/assets/044391af-4b3f-4708-a75a-841f17bb8167)

2. **SSH Key Pair** named `my-ec2-key` to access the instance on port `22`.
3. Configured **security group** with the following inbound rules:
   - Port `80` (HTTP) open to all.
   - Port `443` (HTTPS) open to all.
   - Port `22` (SSH) open to all.
![Screenshot 2024-09-14 112759](https://github.com/user-attachments/assets/c7f80e93-bf36-4f35-b1ee-ee39860fc2da)

4. **Default VPC** and **Subnet** used for networking.
![Screenshot 2024-09-14 113014](https://github.com/user-attachments/assets/7e92e9d2-985f-4300-a04b-813fcd97463c)

## Step 0: Connect to EC2

1. Use the SSH key pair to connect to your EC2 instance:
    ```bash
    chmod 400 my-ec2-key.pem
    ssh -i "my-ec2-key.pem" ubuntu@<your-public-ip>
    ```
![Screenshot 2024-09-14 112021](https://github.com/user-attachments/assets/e8b082ec-c09c-4203-bea4-c6b4ecf717ce)
![Screenshot 2024-09-14 112044](https://github.com/user-attachments/assets/e61bafec-ff1a-47e8-99a6-e83a804ac3d3)

## Step 1: Install Apache and Update the Firewall

1. **Update and upgrade** the package manager:
    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```
![Screenshot 2024-09-14 113423](https://github.com/user-attachments/assets/6228241d-d626-46e8-8114-9eb82a17184a)

2. **Install Apache**:
    ```bash
    sudo apt install apache2 -y
    ```

3. **Enable and check** Apache service:
    ```bash
    sudo systemctl enable apache2
    sudo systemctl status apache2
    ```

![Screenshot 2024-09-14 114435](https://github.com/user-attachments/assets/0cb554b8-5d1f-49c3-abb5-3d582cf1e3a9)

4. Test Apache locally:
    ```bash
    curl http://localhost:80
    ```

5. Test Apache with the public IP:
    ```bash
    http://<your-public-ip>
    ```

![Screenshot 2024-09-14 114718](https://github.com/user-attachments/assets/975a87fb-bd35-43b2-b585-535e668428d3)

## Step 2: Install MySQL

1. **Install MySQL**:
    ```bash
    sudo apt install mysql-server
    ```
![Screenshot 2024-09-14 122633](https://github.com/user-attachments/assets/9e08c60c-f8f7-4632-9a72-d4a3eab09262)

2. **Enable and check** MySQL service:
    ```bash
    sudo systemctl enable --now mysql
    sudo systemctl status mysql
    ```

3. **Log in** to the MySQL console:
    ```bash
    sudo mysql -p
    ```

![Screenshot 2024-09-14 122633](https://github.com/user-attachments/assets/c592f627-9669-4d28-a6e9-fbc2ea05d116)

4. **Set a root password**:
    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
    ```

## Step 3: Install PHP

1. **Install PHP and modules**:
    ```bash
    sudo apt install php libapache2-mod-php php-mysql
    ```
 ![Screenshot 2024-09-14 123541](https://github.com/user-attachments/assets/46e33458-dfde-438a-bdc5-98ecf69c850b)

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
![Screenshot 2024-09-14 124844](https://github.com/user-attachments/assets/2d19158a-3cbd-473d-9f32-8662915454ff)


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
![Screenshot 2024-09-14 125349](https://github.com/user-attachments/assets/ac95b4e2-1749-430f-93c6-077ff8f4b9e4)


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
![Screenshot 2024-09-14 132203](https://github.com/user-attachments/assets/bd155a60-bde7-4c97-83a4-f0e524c740b7)


![Screenshot 2024-09-14 131417](https://github.com/user-attachments/assets/23f66937-7172-403f-a3b7-7b91d2948903)

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
![Screenshot 2024-09-14 132530](https://github.com/user-attachments/assets/a5cca97b-1540-4ce4-8edd-ba8759e19658)


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
![Screenshot 2024-09-14 132819](https://github.com/user-attachments/assets/6c42c237-2fb8-4007-8a3c-be3e0c6da69a)


4. **Access the PHP page**:
    Visit the public IP:
    ```bash
    http://<your-public-ip>
    ```

5. **Remove the test file** for security:
    ```bash
    sudo rm /var/www/projectlamp/index.php
    ```

![Screenshot 2024-09-14 132903](https://github.com/user-attachments/assets/980127fb-8d1b-4256-843d-39fba363434d)

## Conclusion

By following the steps outlined above, you have successfully set up a LAMP stack on AWS EC2. This provides a solid environment for deploying and managing dynamic web applications.

