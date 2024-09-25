# LEMP Stack Implementation in AWS

## Introduction
The LEMP stack is a popular open-source web development platform that consists of four main components: **Linux**, **Nginx**, **MySQL**, and **PHP** (or sometimes Perl or Python). This documentation outlines the setup, configuration, and usage of the LEMP stack on an AWS EC2 instance.

## Prerequisites
- An EC2 instance of type `t2.micro` running **Ubuntu 24.04 LTS (HVM)** launched in the **us-east-1** region using the AWS console.
- An SSH key pair named `my-ec2-key` created to access the instance on port 22.
- Security group configured with the following inbound rules:
  - Allow traffic on port 80 (HTTP) from anywhere on the internet.
  - Allow traffic on port 443 (HTTPS) from anywhere on the internet.
  - Allow traffic on port 22 (SSH) from any IP address (default configuration).
- Default VPC and subnet used for networking configuration.

### SSH Access
1. Change the permissions of the private key file:
   ```bash
   chmod 400 my-ec2-key.pem
   ```
2. Connect to the instance:
   ```bash
   ssh -i "my-ec2-key.pem" ubuntu@18.209.18.61
   ```
   (Replace `18.209.18.61` with your instance's public IP address.)

## Step 1 - Install Nginx Web Server
1. Update and upgrade the server's package index:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```
2. Install Nginx:
   ```bash
   sudo apt install nginx -y
   ```
3. Verify that Nginx is active and running:
   ```bash
   sudo systemctl status nginx
   ```
4. Access Nginx locally:
   ```bash
   curl http://127.0.0.1:80
   ```
5. Test Nginx from a browser using your instance's public IP:
   ```
   http://18.209.18.61:80
   ```

### Troubleshooting 401 Unauthorized Error
To retrieve the public IP address without error, modify the instance metadata options:
- Navigate to **Actions > Instance Settings > Modify instance metadata options**.
- Change IMDSv2 from **Required** to **Optional**.

Then run:
```bash
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

## Step 2 - Install MySQL
1. Install MySQL server:
   ```bash
   sudo apt install mysql-server
   ```
2. Log in to the MySQL console:
   ```bash
   sudo mysql
   ```
3. Set a password for the root user:
   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
   ```
4. Secure MySQL installation:
   ```bash
   sudo mysql_secure_installation
   ```

## Step 3 - Install PHP
1. Install PHP and required extensions:
   ```bash
   sudo apt install php-fpm php-mysql -y
   ```

## Step 4 - Configure Nginx to Use PHP Processor
1. Create a root web directory:
   ```bash
   sudo mkdir /var/www/projectLEMP
   ```
2. Assign ownership of the directory:
   ```bash
   sudo chown -R $USER:$USER /var/www/projectLEMP
   ```
3. Create a new configuration file in Nginx’s `sites-available` directory:
   ```bash
   sudo nano /etc/nginx/sites-available/projectLEMP
   ```
   Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name projectLEMP www.projectLEMP;
       root /var/www/projectLEMP;

       index index.html index.htm index.php;

       location / {
           try_files $uri $uri/ =404;
       }

       location ~ .php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
       }

       location ~ /.ht {
           deny all;
       }
   }
   ```
4. Activate the configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
   ```
5. Test the configuration for syntax errors:
   ```bash
   sudo nginx -t
   ```
6. Disable the default Nginx host:
   ```bash
   sudo unlink /etc/nginx/sites-enabled/default
   ```
7. Reload Nginx:
   ```bash
   sudo systemctl reload nginx
   ```
8. Create a test index.html file:
   ```bash
   echo 'Hello LEMP from hostname $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) with public IP $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)' | sudo tee /var/www/projectLEMP/index.html
   ```
9. Open the website in a browser:
   ```
   http://18.209.18.61:80
   ```

## Step 5 - Test PHP with Nginx
1. Create a test PHP file:
   ```bash
   sudo nano /var/www/projectLEMP/info.php
   ```
   Add the PHP code to query the database (details omitted in the original document).

2. Access the PHP file in a browser:
   ```
   http://18.209.18.61/info.php

   ```

## Step 6 - Retrieve Data from MySQL database with PHP
Create a new user with the mysql_native_password authentication method in order to be able to connect to MySQL database from PHP.

Create a database named todo_database and a user named todo_user

1. First, connect to the MySQL console using the root account.

sudo mysql -p

2. Create a new database

CREATE DATABASE todo_database;

3. Create a new user and grant the user full privileges on the new database.

CREATE USER 'todo_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Admin123$';

GRANT ALL ON todo_database.* TO 'todo_user'@'%';

exit
4. Login to MySQL console with the user custom credentials and confirm that you have access to todo_database.

mysql -u todo_user -p

SHOW DATABASES;

The -p flag will prompt for password used when creating the example_user

5. Create a test table named todo_list.

From MySQL console, run the following:

CREATE TABLE todo_database.todo_list (
  item_id INT AUTO_INCREMENT,
  content VARCHAR(255),
  PRIMARY KEY(item_id)
);
6. Insert a few rows of content to the test table.

INSERT INTO todo_database.todo_list (content) VALUES ("My first important item");

INSERT INTO todo_database.todo_list (content) VALUES ("My second important item");

INSERT INTO todo_database.todo_list (content) VALUES ("My third important item");

INSERT INTO todo_database.todo_list (content) VALUES ("and this one more thing");

7. To confirm that the data was successfully saved to the table run:

SELECT * FROM todo_database.todo_list; 

exit
Create a PHP script that will connect to MySQL and query the content.
1. Create a new PHP file in the custom web root directory

sudo nano /var/www/projectLEMP/todo_list.php
The PHP script connects to MySQL database and queries for the content of the todo_list table, displays the results in a list. If there’s a problem with the database connection, it will throw an exception.

Copy the content below into the todo_list.php script.

<?php
$user = "todo_user";
$password = "Admin123$";
$database = "todo_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>

2. Now access this page on the browser by using the domain name or public IP address followed by /todo_list.php

http://18.209.18.61/todo_list.php

2. Now access this page on the browser by using the domain name or public IP address followed by /todo_list.php

http://18.209.18.61/todo_list.php

Ater updating the Nginx server, the URL was tested again on the browser and there no error.

http://18.209.18.61/todo_list.php

Access this page on the browser by using the domain name followed by /todo_list.php

Conclusion
The LEMP stack provides a robust platform for hosting and serving web applications. By leveraging the power of Linux, Nginx, MySQL (or MariaDB), and PHP, developers can deploy scalable and reliable web solutions.

### Troubleshooting PHP Errors
If you encounter a "502 Bad Gateway" error, ensure the PHP version in the Nginx configuration matches the installed version. Update to the correct version as needed.

## Conclusion
The LEMP stack provides a robust platform for hosting and serving web applications. By leveraging the power of Linux, Nginx, MySQL, and PHP, developers can deploy scalable and reliable web solutions.
