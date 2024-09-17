WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

Introduction:
The LAMP stack is a widely-used open-source web development platform, known for its reliability and flexibility. It comprises four key components: Linux (the operating system), Apache (the web server), MySQL (the database management system), and PHP, Perl, or Python (the programming languages). Each component plays a crucial role in creating a fully functional web application. This documentation provides detailed guidance on setting up, configuring, and utilizing the LAMP stack, allowing developers to build dynamic websites and applications efficiently. It covers installation procedures, configuration tips, and best practices for optimizing performance and security.

STEP 0: Prerequisites

1: EC2 Instance of t2.micro type and Ubuntu 24.04 LTS (HVM) was lunched in the us-east-1 region using the AWS console.


![Screenshot 2024-09-14 111739](https://github.com/user-attachments/assets/9391bb31-dcb2-4a67-9884-001b3afc3081)
![Screenshot 2024-09-14 111938](https://github.com/user-attachments/assets/e9702aa4-3799-4ce9-af66-222f5b295ed8)

2. Created SSH key pair named my-ec2-key to access the instance on port 22.
3.  The security group was configured with the following inbound rules:
  - Allow traffic on port 80 (HTTP) with source from anywhere on the internet.
  - Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.
  - Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.

![Screenshot 2024-09-14 112759](https://github.com/user-attachments/assets/a7f792cf-25bf-4fb5-8319-2b2208bf6424)
4. The default VPC and Subnet was used for the networking configuration.
![Screenshot 2024-09-14 113014](https://github.com/user-attachments/assets/8d203e37-44f2-4746-81a3-af373b2dd5ec)

5. The private ssh key that got downloaded was located, permission was changed for the private key file and then used to connect to the instance by running

chmod 400 my-ec2-key.pem
ssh -i "my-ec2-key.pem" ubuntu@3.84.147.176
Where username=ubuntu and public ip address=3.84.147.176
![Screenshot 2024-09-14 112021](https://github.com/user-attachments/assets/ca7788a0-3b4c-49ce-8bd6-e4f777a3fb90)
![Screenshot 2024-09-14 112044](https://github.com/user-attachments/assets/04f052fd-9b69-41a2-94eb-2d7978f06595)

Step 1 - Install Apache and Update the Firewall
1. Update and upgrade list of packages in package manager.
   sudo apt update
   sudo apt upgrade -y
![Screenshot 2024-09-14 113423](https://github.com/user-attachments/assets/2435a41a-05b6-4eed-b8ef-38a7c3411043)

2. Run apache2 package installation
  sudo apt install apache2 -y
3. Enable and verify that apache is running on as a service on the OS.
   sudo systemctl enable apache2
   sudo systemctl status apache2
If it green and running, then apache2 is correctly installed
![Screenshot 2024-09-14 114435](https://github.com/user-attachments/assets/cc3f0b26-aa3c-4ea8-8c01-cd1a9ab3f915)

4. The server is running and can be accessed locally in the ubuntu shell by running the command below:
   curl http://localhost:80
  OR
  curl http://127.0.0.1:80
5. Test with the public IP address if the Apache HTTP server can respond to request from the internet using the url on a browser.
 http://3.84.147.176
![Screenshot 2024-09-14 114639](https://github.com/user-attachments/assets/02483603-b2d0-4f22-9fd1-03a98a8e9a21)
This shows that the web server is correctly installed and it is accessible through the firewall.

Step 2 - Install MySQL
1. Install a relational database (RDB)

MySQL was installed in this project. It is a popular relational database management system used within PHP environments.
  sudo apt install mysql-server
When prompted, install was confirmed by typing y and then Enter.

2. Enable and verify that mysql is running with the commands below.
   sudo systemctl enable --now mysql
   sudo systemctl status mysql

3.  Log in to mysql console
   sudo mysql -p
This connects to the MySQL server as the administrative database user root infered by the use of sudo when running the command.

4. Set a password for root user using mysql_native_password as default authentication method.

Here, the user's password was defined as "Admin123$"
  ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
![Screenshot 2024-09-14 122633](https://github.com/user-attachments/assets/a0e6b3bf-b4d7-4732-8519-5473cc04ef23)

Step 3 - Install PHP
1. Install php Apache is installed to serve the content and MySQL is installed to store and manage data. PHP is the component of the set up that processes code to display dynamic content to the end user.

The following were installed:
php package
php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.
libapache2-mod-php, to enable Apache to handle PHP files.
  sudo apt install php libapache2-mod-php php-mysql.
  ![Screenshot 2024-09-14 123541](https://github.com/user-attachments/assets/dae0af51-1a48-4668-97c8-fbaa5bf929a0)
Confirm the PHP version
php -v

Step 4 - Create a virtual host for the website using Apache
1. The default directory serving the apache default page is /var/www/html. Create your document directory next to the default one.

Created the directory for projectlamp using "mkdir" command
  sudo mkdir /var/www/projectlamp
Assign the directory ownership with $USER environment variable which references the current system user.
  sudo chown
![Screenshot 2024-09-14 124844](https://github.com/user-attachments/assets/f7b65c4d-c348-4f83-8502-86c707ab6950)
 -R $USER:$USER /var/www/projectlamp

2. Create and open a new configuration file in apache’s “sites-available” directory using vim.
   sudo vim /etc/apache2/sites-available/projectlamp.conf
Paste in the bare-bones configuration below:
<VirtualHost *:80>
  ServerName projectlamp
  ServerAlias www.projectlamp
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/projectlamp
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

![Screenshot 2024-09-14 125349](https://github.com/user-attachments/assets/2afaee9c-9f44-4474-a2ae-7949bd9aca78)

3. Show the new file in sites-available
  sudo ls /etc/apache2/sites-available
Output:
000-default.conf default-ssl.conf projectlamp.conf

With the VirtualHost configuration, Apache will serve projectlamp using /var/www/projectlamp as its web root directory.

4. Enable the new virtual host

sudo a2ensite projectlamp

5. Disable apache’s default website.

This is because Apache’s default configuration will overwrite the virtual host if not disabled. This is required if a custom domain is not being used.

sudo a2dissite 000-default

6. Ensure the configuration does not contain syntax error

The command below was used:

sudo apache2ctl configtest

7. Reload apache for changes to take effect.

sudo systemctl reload apache2
![Screenshot 2024-09-14 132203](https://github.com/user-attachments/assets/9f34dfb5-4443-4eaa-a21b-0f1c17ae7b6c)


8. The new website is now active but the web root /var/www/projectlamp is still empty. Create an index.html file in this location so to test the virtual host work as expected.

9. Open the website on a browser using the public IP address.

http://3.84.147.176
![Screenshot 2024-09-14 131417](https://github.com/user-attachments/assets/94c1fc99-77ac-49c2-ac43-ffa9ae745e77)

10. Open the website with public dns name (port is optional)

http://<public-DNS-name>:80

![Screenshot 2024-09-14 131417](https://github.com/user-attachments/assets/619bc5a0-452e-4325-af2e-5998cff5711c)

This file can be left in place as a temporary landing page for the application until an index.php file is set up to replace it. Once this is done, the index.html file should be renamed or removed from the document root as it will take precedence over index.php file by default.

Step 5 - Enable PHP on the website
With the default DirectoryIndex setting on Apache, index.html file will always take precedence over index.php file. This is useful for setting up maintenance page in PHP applications, by creating a temporary index.html file containing an informative message for visitors. The index.html then becomes the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root bringing back the regular application page. If the behaviour needs to be changed, /etc/apache2/mods-enabled/dir.conf file should be edited and the order in which the index.php file is listed within the DirectoryIndex directive should be changed.

1. Open the dir.conf file with vim to change the behaviour
  sudo vim /etc/apache2/mods-enabled/dir.conf
  <IfModule mod_dir.c>
  # Change this:
  # DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
  # To this:
  DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>

![Screenshot 2024-09-14 132530](https://github.com/user-attachments/assets/944e8f26-6a2d-4c8d-95cf-a5364e39432c)

2. Reload Apache

Apache is reloaded so the changes takes effect.

sudo systemctl reload apache2

3. Create a php test script to confirm that Apache is able to handle and process requests for PHP files.

A new index.php file was created inside the custom web root folder.

vim /var/www/projectlamp/index.php
Add the text below in the index.php file

<?php
phpinfo();

![Screenshot 2024-09-14 132819](https://github.com/user-attachments/assets/357c441e-2b96-45ae-aa95-199d242147b7)
4. Now refresh the page

![Screenshot 2024-09-14 132903](https://github.com/user-attachments/assets/baadcf1b-e7c4-4310-955d-60b55971b7ea)


This page provides information about the server from the perspective of PHP. It is useful for debugging and to ensure the settings are being applied correctly.

After checking the relevant information about the server through this page, It’s best to remove the file created as it contains sensitive information about the PHP environment and the ubuntu server. It can always be recreated if the information is needed later.

sudo rm /var/www/projectlamp/index.php
Conclusion:

The LAMP stack provides a robust and flexible platform for developing and deploying web applications. By following the guidelines outlined in this documentation, It was possible to set up, configure, and maintain a LAMP environment effectively, enabling the creation of powerful and scalable web solutions.

![Screenshot 2024-09-14 134423](https://github.com/user-attachments/assets/8747c328-b80d-47f6-ba03-0184adb1562e)
