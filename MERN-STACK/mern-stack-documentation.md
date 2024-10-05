The LEMP stack is a widely-used open-source platform that consists of four essential components: Linux, Nginx, MySQL, and PHP (or at times, Perl or Python). This documentation presents the steps involved in setting up, configuring, and utilizing the LEMP stack within an AWS environment.

Step 0: Prerequisites
EC2 Instance: Launched a t2.micro EC2 instance running Ubuntu 24.04 LTS (HVM) in the us-east-1 region via the AWS console.
SSH Key Pair: Created a key pair named my-ec2-key for accessing the instance through port 22.
Security Group Configuration:
Allow HTTP traffic on port 80 from any source.
Allow HTTPS traffic on port 443 from any source.
Allow SSH traffic on port 22 from any IP address.
Network Configuration: Utilized the default VPC and subnet for networking.
Instance Access: Modified private SSH key permissions and used it to connect to the instance using the command:
bash
Copy code
chmod 400 my-ec2-key.pem
ssh -i "my-ec2-key.pem" ubuntu@18.209.18.61
Step 1: Install Nginx Web Server
Update and Upgrade the package index:
bash
Copy code
sudo apt update
sudo apt upgrade -y
Install Nginx:
bash
Copy code
sudo apt install nginx -y
Check Nginx Status to ensure it is active and running:
bash
Copy code
sudo systemctl status nginx
Local Access: Test Nginx locally using:
bash
Copy code
curl http://127.0.0.1:80
Public Access: Test public access using the browser URL:
arduino
Copy code
http://18.209.18.61:80
Retrieve Public IP:
If a "401 - Unauthorized" error occurs, modify EC2 instance metadata settings to allow IMDSv1 by changing IMDSv2 from "Required" to "Optional."
Then retrieve the public IP:
bash
Copy code
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
Step 2: Install MySQL
Install MySQL:
bash
Copy code
sudo apt install mysql-server
Access MySQL Console:
bash
Copy code
sudo mysql
Set Root Password:
bash
Copy code
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
Run Security Script:
bash
Copy code
sudo mysql_secure_installation
Step 3: Install PHP
Install PHP-FPM and MySQL Extension:
bash
Copy code
sudo apt install php-fpm php-mysql -y
Step 4: Configure Nginx for PHP
Create Root Directory for your website:
bash
Copy code
sudo mkdir /var/www/projectLEMP
sudo chown -R $USER:$USER /var/www/projectLEMP
Create Nginx Configuration:
bash
Copy code
sudo nano /etc/nginx/sites-available/projectLEMP
Add the following configuration:
perl
Copy code
server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;
    index index.html index.htm index.php;
    location / {
        try_files $uri $uri/ =404;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }
    location ~ /\.ht {
        deny all;
    }
}
Activate the Configuration:
bash
Copy code
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
sudo nginx -t
sudo unlink /etc/nginx/sites-enabled/default
sudo systemctl reload nginx
Step 5: Test PHP with Nginx
Create Test PHP File:
bash
Copy code
sudo nano /var/www/projectLEMP/info.php
Add the following content:
php
Copy code
<?php
phpinfo();
?>
Access via Browser using:
arduino
Copy code
http://18.209.18.61/info.php
Step 6: Retrieve Data from MySQL with PHP
Create MySQL Database:
bash
Copy code
sudo mysql -p
CREATE DATABASE todo_database;
CREATE USER 'todo_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
GRANT ALL ON todo_database.* TO 'todo_user'@'%';
Create PHP Script:
bash
Copy code
sudo nano /var/www/projectLEMP/todo_list.php
Insert:
php
Copy code
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
Conclusion
With the successful configuration of the LEMP stack, developers can now host scalable web applications powered by Linux, Nginx, MySQL, and PHP.
