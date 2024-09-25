![Screenshot 2024-09-23 152403](https://github.com/user-attachments/assets/2641530b-0c44-468b-8c52-2ed3b416f5d2)# LEMP Stack Implementation in AWS

## Introduction
The LEMP stack is a popular open-source web development platform that consists of four main components: **Linux**, **Nginx**, **MySQL**, and **PHP** (or sometimes Perl or Python). This documentation outlines the setup, configuration, and usage of the LEMP stack on an AWS EC2 instance.

## Step 0: Prerequisites
- An EC2 instance of type `t2.micro` running **Ubuntu 24.04 LTS (HVM)** launched in the **us-east-1** region using the AWS console.
![Screenshot 2024-09-23 151110](https://github.com/user-attachments/assets/0d9dfcba-b451-4c97-93d8-fd77e9b855e1)

![Screenshot 2024-09-23 151133](https://github.com/user-attachments/assets/335f4584-3565-4db8-8fb9-e16c3942c6aa)



- An SSH key pair named `my-ec2-key` created to access the instance on port 22.
- Security group configured with the following inbound rules:
  - Allow traffic on port 80 (HTTP) from anywhere on the internet.
  - Allow traffic on port 443 (HTTPS) from anywhere on the internet.
  - Allow traffic on port 22 (SSH) from any IP address (default configuration).
![Screenshot 2024-09-23 151320](https://github.com/user-attachments/assets/4eaa86c9-1ea4-476e-aed1-4c9a1786b809)

- Default VPC and subnet used for networking configuration.
![Screenshot 2024-09-23 151241](https://github.com/user-attachments/assets/b7d7513e-23e6-4fd1-81eb-6c2fe2a4c681)

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

![Screenshot 2024-09-23 151554](https://github.com/user-attachments/assets/1f656567-9c13-44dd-b29d-e0217b522e3c)


## Step 1 - Install Nginx Web Server
1. Update and upgrade the server's package index:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```
   ![Screenshot 2024-09-23 152159](https://github.com/user-attachments/assets/fee55d76-4d20-4149-bf4d-430482f9a051)

2. Install Nginx:
   ```bash
   sudo apt install nginx -y
   ```
![Screenshot 2024-09-23 152403](https://github.com/user-attachments/assets/58e42762-3b37-4a31-937e-eb3b9e779db4)


  
3. Verify that Nginx is active and running:
   ```bash
   sudo systemctl status nginx
   ```
![Screenshot 2024-09-23 152504](https://github.com/user-attachments/assets/8f161617-5d89-4965-94cc-6d532029f8e2)
   
4. Access Nginx locally:
   ```bash
   curl http://127.0.0.1:80
   ```
![Screenshot 2024-09-23 152654](https://github.com/user-attachments/assets/14f41011-3280-4ea8-b630-cc80783aced1)

   
5. Test Nginx from a browser using your instance's public IP:
   ```
   http://18.209.18.61:80
   ```
   ![Screenshot 2024-09-23 152826](https://github.com/user-attachments/assets/081a9383-e2cd-4e05-b046-bd78ad3e76c3)



### Troubleshooting 401 Unauthorized Error
![Screenshot 2024-09-23 153740](https://github.com/user-attachments/assets/045fbf68-f864-40ac-9d6b-768aea6bdfbb)


To retrieve the public IP address without error, modify the instance metadata options:
- Navigate to **Actions > Instance Settings > Modify instance metadata options**.
- Change IMDSv2 from **Required** to **Optional**.
![Screenshot 2024-09-23 153147](https://github.com/user-attachments/assets/1d9e66d2-288d-459a-9029-6b5a7d9bc795)

Then run:
```bash
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

## Step 2 - Install MySQL
1. Install MySQL server:
   ```bash
   sudo apt install mysql-server
   ```
![Screenshot 2024-09-23 153952](https://github.com/user-attachments/assets/f70f0e85-3714-4acf-9b49-d8b3cc281678)

   
2. Log in to the MySQL console:
   ```bash
   sudo mysql
   ```
![Screenshot 2024-09-23 154040](https://github.com/user-attachments/assets/730a949b-0edb-43d5-b044-7d7e0eb1c3db)

   
3. Set a password for the root user:
   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
   ```
4. Secure MySQL installation:
   ```bash
   sudo mysql_secure_installation
   ```
![Screenshot 2024-09-23 154711](https://github.com/user-attachments/assets/7cc26171-04e9-4b7f-95f6-6b459e169990)

## Step 3 - Install PHP
1. Install PHP and required extensions:
Install php-fpm (PHP fastCGI process manager) and tell nginx to pass PHP requests to this software for processing. Also, install php-mysql, a php module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

The following were installed:
php-fpm (PHP fastCGI process manager)
php-mysql

   ```bash
   sudo apt install php-fpm php-mysql -y
   ```

   ![Screenshot 2024-09-23 155845](https://github.com/user-attachments/assets/74df2ba8-1882-4a02-b707-09f83bc460b3)


## Step 4 - Configure Nginx to Use PHP Processor
1. Create a root web directory:
   ```bash
   sudo mkdir /var/www/projectLEMP
   ```
![Screenshot 2024-09-23 160304](https://github.com/user-attachments/assets/0e71a06f-6334-44fe-8e4c-29bd5f7fa3e4)

   
2. Assign ownership of the directory:
   ```bash
   sudo chown -R $USER:$USER /var/www/projectLEMP
   ```
![Screenshot 2024-09-23 160304](https://github.com/user-attachments/assets/a2d0444f-777a-4e4c-ab6d-00554481a2c6)

   
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
   
![Screenshot 2024-09-23 161543](https://github.com/user-attachments/assets/1e7e9293-0bfa-438a-9af6-8d4243880c84)

Here’s what each directives and location blocks does:
listen - Defines what port nginx listens on. In this case it will listen on port 80, the default port for HTTP.

root - Defines the document root where the files served by this website are stored.

index - Defines in which order Nginx will prioritize the index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page for PHP applications. You can adjust these settings to better suit your application needs.

server_name - Defines which domain name and/or IP addresses the server block should respond for. Point this directive to your domain name or public IP address.

location / - The first location block includes the try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate result, it will return a 404 error.

location ~ .php$ - This location handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

location ~ /.ht - The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root, they will not be served to visitors.

   
4. Activate the configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
   ```
   
![Screenshot 2024-09-23 162029](https://github.com/user-attachments/assets/38142cc3-4a27-4444-97d1-ddf5f4f8fa40)

   
5. Test the configuration for syntax errors:
   ```bash
   sudo nginx -t
   ```
   ![Screenshot 2024-09-23 162232](https://github.com/user-attachments/assets/12662031-542c-42f3-a344-9a933b464dce)

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
