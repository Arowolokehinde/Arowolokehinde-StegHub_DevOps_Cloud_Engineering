# DevOps Tooling Website Solution

## Introduction

This project involves implementation of a solution that consists of the following components:

- **Infrastructure:** AWS
- **Web Server Linux:** Red Hat Enterprise Linux 9
- **Database Server:** Ubuntu Linux + MySQL
- **Storage Server:** Red Hat Enterprise Linux 9 + NFS Server
- **Programming Language:** PHP
- **Code Repository:** GitHub

The diagram below shows the architecture of the solution.

3 tier

## Step 1 - Prepare NFS Server

1. Spin up an EC2 instance with RHEL Operating System

![Screenshot 2024-10-29 091833](https://github.com/user-attachments/assets/ac6bff73-a016-4de8-b260-145f62cb2842)


2. Configure Logical volume management on the server

    - Format the lvm as xfs
    - Create 3 Logical volumes: lv-opt, lv-appa, lv-logs.
    - Create mount points on /mnt directory for the logical volumes as follows:
      - Mount lv-apps on /mnt/apps - To be used by web servers
      - Mount lv-logs on /mnt/logs - To be used by web server logs
      - Mount lv-opt on /mnt/opt - To be used by Jenkins server in next project.
    - Create 3 volumes in the same AZ as the NFS Server ec2 each of 10GB and attach all 3 volumes one by one to the NFS Server.

![Screenshot 2024-10-29 091923](https://github.com/user-attachments/assets/659cbd16-dd31-4bc2-ae83-2e2435631bb6)
    Open up the Linux terminal to begin configuration.
    ```bash
    ssh -i "my-devec2key.pem" ec2-user@18.224.38.96
    ```
    ![Screenshot 2024-10-29 091822](https://github.com/user-attachments/assets/6495e310-fa45-47f0-89ae-b2a346dc2519)


 Use `lsblk` to inspect what block devices are attached to the server. All devices in Linux reside in `/dev/` directory. Inspect with `ls /dev/` and ensure all 3 newly created devices are there. Their name will likely be xvdf, xvdg and xvdh
    ```bash
    lsblk
    ```
![Screenshot 2024-10-29 092415](https://github.com/user-attachments/assets/0ed79248-be03-44a2-808d-fe20401cf022)


 Use `gdisk` utility to create a single partition on each of the 3 disks
    ```bash
    sudo gdisk /dev/xvdf
    ```
    ![Screenshot 2024-10-29 092427](https://github.com/user-attachments/assets/6d11bbd3-cf46-46b1-b537-3606df5a8ac7)

 ```bash
    sudo gdisk /dev/xvdg
 ```
  ![Screenshot 2024-10-29 092438](https://github.com/user-attachments/assets/930d8b14-6dd4-48e0-93d5-c5e3fc28524a)


```bash
    sudo gdisk /dev/xvdh
```
![Screenshot 2024-10-29 092507](https://github.com/user-attachments/assets/59173f1e-3648-427e-9f71-c76bbe96bb32)



Use `lsblk` utility to view the newly configured partitions on each of the 3 disks
    ```bash
    lsblk
    ```
   ![Screenshot 2024-10-29 092528](https://github.com/user-attachments/assets/c41ee9f6-04f8-4a8e-8e49-43d8d37022d3)


  Install lvm package
    ```bash
    sudo yum install lvm2 -y
    ```
  
![Screenshot 2024-10-29 092653](https://github.com/user-attachments/assets/cae745e7-bb47-4f09-b6cd-9b5a097fef66)
![Screenshot 2024-10-29 092724](https://github.com/user-attachments/assets/23f0e432-cabd-4746-82ab-effd34355bbf)


 Use `pvcreate` utility to mark each of the 3 disks as physical volumes (PVs) to be used by LVM. Verify that each of the volumes have been created successfully
    ```bash
    sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo pvs
    ```
![Screenshot 2024-10-29 093319](https://github.com/user-attachments/assets/7b8447ad-1ca3-446b-bf8e-a7d9121e6157)


 Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG `webdata-vg`. Verify that the VG has been created successfully
    ```bash
    sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo vgs
    ```
![Screenshot 2024-10-29 093331](https://github.com/user-attachments/assets/b5d07255-fc4d-4aeb-b5c8-ee5e296f9568)


 Use `lvcreate` utility to create 3 logical volumes, lv-apps, lv-logs and lv-opt. Verify that the logical volumes have been created successfully
    ```bash
    sudo lvcreate -n lv-apps -L 8G webdata-vg
    sudo lvcreate -n lv-logs -L 8G webdata-vg
    sudo lvcreate -n lv-opt -L 8G webdata-vg
    sudo lvs
    ```
![Screenshot 2024-10-29 093339](https://github.com/user-attachments/assets/5ea4c081-98dd-43f8-9d24-c7f7738e1fef)

 Verify the entire setup
    ```bash
    sudo vgdisplay -v   #view complete setup, VG, PV and LV
    lsblk
    ```
 ![Screenshot 2024-10-29 093422](https://github.com/user-attachments/assets/262ad914-e05e-4767-af4a-a9ddb2fdf499)
![Screenshot 2024-10-29 093433](https://github.com/user-attachments/assets/ba280798-bd90-4644-a8e7-cbe21ca59cd2)


 Use `mkfs -t xfs` to format the logical volumes instead of ext4 filesystem
    ```bash
    sudo mkfs -t xfs /dev/webdata-vg/lv-apps
    sudo mkfs -t xfs /dev/webdata-vg/lv-logs
    sudo mkfs -t xfs /dev/webdata-vg/lv-opt
    ```
![Screenshot 2024-10-29 093619](https://github.com/user-attachments/assets/9586a55f-4459-4b8f-bff0-fffcaccb113c)

  Create mount point on /mnt directory
    ```bash
    sudo mkdir /mnt/apps
    sudo mkdir /mnt/logs
    sudo mkdir /mnt/opt
    sudo mount /dev/webdata-vg/lv-apps /mnt/apps
    sudo mount /dev/webdata-vg/lv-logs /mnt/logs
    sudo mount /dev/webdata-vg/lv-opt /mnt/opt
    ```
![Screenshot 2024-10-29 093628](https://github.com/user-attachments/assets/7e97ebf8-5739-4963-bcfd-5668430901f7)
![Screenshot 2024-10-29 094437](https://github.com/user-attachments/assets/6920bb6b-f8a6-498b-b2e0-11bd16769f8b)
![Screenshot 2024-10-29 095248](https://github.com/user-attachments/assets/94c1e858-b7bc-4e02-9f2c-4c6090b8648b)
![Screenshot 2024-10-29 094451](https://github.com/user-attachments/assets/6a392ed9-3594-4363-8c47-b72b4df822f8)


3. Install NFS Server, configure it to start on reboot and ensure it is up and running.
    ```bash
    sudo yum update -y
    sudo yum install nfs-utils -y
    sudo systemctl start nfs-server.service
    sudo systemctl enable nfs-server.service
    sudo systemctl status nfs-server.service
    ```
    ![Screenshot 2024-10-29 095818](https://github.com/user-attachments/assets/1a5a3ee8-f7cc-423b-9296-dbd00472fdc9)

![Screenshot 2024-10-29 100143](https://github.com/user-attachments/assets/413b45ef-2ec8-4572-a0c3-f4758bb06fc6)
![Screenshot 2024-10-29 100208](https://github.com/user-attachments/assets/0cae49fd-dfd2-4f39-a5b7-2418cf478392)

4. Export the mounts for Webservers' subnet cidr(IPv4 cidr) to connect as clients. For simplicity, all 3 Web Servers are installed in the same subnet but in production set up, each tier should be separated inside its own subnet or higher level of security

    Set up permission that will allow the Web Servers to read, write and execute files on NFS.
    ```bash
    sudo chown -R nobody: /mnt/apps
    sudo chown -R nobody: /mnt/logs
    sudo chown -R nobody: /mnt/opt
    sudo chmod -R 777 /mnt/apps
    sudo chmod -R 777 /mnt/logs
    sudo chmod -R 777 /mnt/opt
    sudo systemctl restart nfs-server.service
    ```
  ![Screenshot 2024-10-29 101255](https://github.com/user-attachments/assets/39d79c69-da6b-415f-aea1-f4f8cdf3956c)


  Configure access to NFS for clients within the same subnet (example Subnet Cidr - 172.31.32.0/20)
    ```bash
    sudo vi /etc/exports
    ```
    ```
    /mnt/apps 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
    /mnt/logs 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
    /mnt/opt 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
    ```
    ```bash
    sudo exportfs -arv
    ```
![Screenshot 2024-10-29 101428](https://github.com/user-attachments/assets/cb196f43-71de-4884-a235-ae129c8434e4)


5. Check which port is used by NFS and open it using the security group (add new inbound rule)
    ```bash
    rpcinfo -p | grep nfs
    ```
![Screenshot 2024-10-29 102022](https://github.com/user-attachments/assets/77295ab1-c6c8-4daa-8edb-70cc5f96fcad)


Note: For NFS Server to be accessible from the client, the following ports must be opened: TCP 111, UDP 111, UDP 2049, NFS 2049. Set the Web Server subnet cidr as the source

![Screenshot 2024-10-29 102002](https://github.com/user-attachments/assets/b4e7cbdd-1d87-4cd2-bbe6-c99c68126d20)


## Step 2 - Configure the Database Server

Launch an Ubuntu EC2 instance that will have a role - DB Server
ec2 db detail

Access the instance to begin configuration.
```bash
ssh -i "my-devec2key.pem" ubuntu@34.235.145.155
```
![Screenshot 2024-10-29 102146](https://github.com/user-attachments/assets/603f122e-8cd7-4121-869e-639fdfe88307)


Update and upgrade Ubuntu
```bash
sudo apt update && sudo apt upgrade -y
```
![Screenshot 2024-10-29 102345](https://github.com/user-attachments/assets/709b2a98-679c-4828-9c90-b8d4bce88efc)


1. Install MySQL Server

    Install mysql server
    ```bash
    sudo apt install mysql-server
    ```
![Screenshot 2024-10-29 102848](https://github.com/user-attachments/assets/cb734588-7b64-4c62-8710-047668eee94c)
    Run mysql secure script
    ```bash
    sudo mysql_secure_installation
    ```

![Screenshot 2024-10-29 102914](https://github.com/user-attachments/assets/0c216b65-5244-4db6-a277-84dcfacf374d)


2. Create a database and name it tooling

3. Create a database user and name it webaccess

4. Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr
    ```bash
    sudo mysql
    ```
    ```sql
    CREATE DATABASE tooling;
    CREATE USER 'webaccess'@'172.31.0.0/20' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
    GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.0.0/20' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    show databases;
    use tooling;
    select host, user from mysql.user;
    exit
    ```
![Screenshot 2024-10-29 102914](https://github.com/user-attachments/assets/7fdd0d8b-b074-4b98-87be-13ceeb4a4fae)

 Set Bind Address and restart MySQL
    ```bash
    sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
    sudo systemctl restart mysql
    sudo systemctl status mysql
    ```
![Screenshot 2024-10-29 103103](https://github.com/user-attachments/assets/298a6d5d-39f6-4ecd-b246-1543a094631e)
    Open MySQL port 3306 on the DB Server EC2. Access to the DB Server is allowed only from the Subnet Cidr configured as source.
![Screenshot 2024-10-29 103114](https://github.com/user-attachments/assets/77f3da4d-6734-45da-a5c0-56a9653788a5)


## Step 3 - Prepare the Web Servers

There is need to ensure that the Web Servers can serve the same content from a shared storage solution, in this case - NFS and MySQL database. One DB can be accessed for read and write by multiple clients. For storing shared files that the Web Servers will use, NFS is utilized and previously created Logical Volume lv-apps is mounted to the folder where Apache stores files to be served to the users (/var/www).

This approach makes the Web server stateless which means they can be replaced when needed and data (in the database and on NFS) integrity is preserved

In further steps, the following was done:

- Configured NFS (This step was done on all 3 servers)
- Deployed a tooling application to the Web Servers into a shared NFS folder
- Configured the Web Server to work with a single MySQL database

### Web Server 1

1. Launch a new EC2 instance with RHEL Operating System

![Screenshot 2024-10-29 103230](https://github.com/user-attachments/assets/8a74ae9d-d83b-4a19-9902-9b841318aab1)
![Screenshot 2024-10-29 103605](https://github.com/user-attachments/assets/8487f00d-de35-4bbc-a6cb-87b59990b223)


2. Install NFS Client
    ```bash
    sudo yum install nfs-utils nfs4-acl-tools -y
    ```
![Screenshot 2024-10-29 124106](https://github.com/user-attachments/assets/8caf3ae9-b96c-4fa1-895c-135e6b4a9ca6)


3. Mount /var/www/ and target the NFS server's export for apps. NFS Server private IP address = 172.31.1.209
    ```bash
    sudo mkdir /var/www
    sudo mount -t nfs -o rw,nosuid 172.31.1.209:/mnt/apps /var/www
    ```
![Screenshot 2024-10-29 124124](https://github.com/user-attachments/assets/6e20e8da-2565-48b4-9c31-795aca32562c)

4. Verify that NFS was mounted successfully by running `df -h`. Ensure that the changes will persist after reboot.

    mount app

    ```bash
    sudo vi /etc/fstab
    ```
    Add the following line
    ```
    172.31.1.209:/mnt/apps /var/www nfs defaults 0 0
    ```
![Screenshot 2024-10-29 124124](https://github.com/user-attachments/assets/b5abedf1-7fc5-45f3-a35e-0ff3084c64db)


5. Install Remi's repository, Apache and PHP
    ```bash
    sudo yum install httpd -y
    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    sudo dnf module reset php
    sudo dnf module enable php:remi-8.2
    sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    sudo systemctl status php-fpm
    sudo setsebool -P httpd_execmem 1  # Allows the Apache HTTP server (httpd) to execute memory that it can also write to. This is often needed for certain types of dynamic content and applications that may need to generate and execute code at runtime.
    sudo setsebool -P httpd_can_network_connect=1   # Allows the Apache HTTP server to make network connections to other servers.
    sudo setsebool -P httpd_can_network_connect_db=1  # allows the Apache HTTP server to connect to remote database servers.
    ```
![Screenshot 2024-10-29 124339](https://github.com/user-attachments/assets/6b25e64f-3842-47e8-aa08-f74abecdec53)
![Screenshot 2024-10-29 124350](https://github.com/user-attachments/assets/f767aee6-fdc2-4900-9e2f-b2c810cffe76)
![Screenshot 2024-10-29 124722](https://github.com/user-attachments/assets/6a91541c-c11e-4765-9ba9-3da44dba6ae3)
![Screenshot 2024-10-29 125213](https://github.com/user-attachments/assets/401f95d3-15e6-464c-b1e4-698e408a4271)
![Screenshot 2024-10-29 125307](https://github.com/user-attachments/assets/499b7c4e-aad3-4e83-b6d1-60ee098ea9c1)


### Web Server 2

1. Launch another new EC2 instance with RHEL Operating System
![Screenshot 2024-10-29 131318](https://github.com/user-attachments/assets/2a7a4a7c-a17c-4e64-b0fd-326f44373d12)


2. Install NFS Client
    ```bash
    sudo yum install nfs-utils nfs4-acl-tools -y
    ```
![Screenshot 2024-10-29 131934](https://github.com/user-attachments/assets/55322e27-5aba-468d-9b20-bd21dfc1c71e)


3. Mount /var/www/ and target the NFS server's export for apps. NFS Server private IP address = 172.31.20.140
    ```bash
    sudo mkdir /var/www
    sudo mount -t nfs -o rw,nosuid 172.31.1.209:/mnt/apps /var/www
    ```
    ![Screenshot 2024-10-29 131551](https://github.com/user-attachments/assets/7479b8a7-52da-4cdd-a539-b0ec908e7144)

4. Verify that NFS was mounted successfully by running `df -h`. Ensure that the changes will persist after reboot.
    ```bash
    sudo vi /etc/fstab
    ```
    Add the following line
    ```
    172.31.1.209:/mnt/apps /var/www nfs defaults 0 0
    ```
![Screenshot 2024-10-29 131551](https://github.com/user-attachments/assets/38bac63a-10d5-4855-8611-eca279fa0117)

![Screenshot 2024-10-29 131527](https://github.com/user-attachments/assets/cba9ec2c-c883-4b37-9604-70cae7ab520f)

5. Install Remi's repository, Apache and PHP
    ```bash
    sudo yum install httpd -y
    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    sudo dnf module reset php
    sudo dnf module enable php:remi-8.2
    sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    sudo systemctl status php-fpm
    sudo setsebool -P httpd_execmem 1
    ```
   ![Screenshot 2024-10-29 131934](https://github.com/user-attachments/assets/ba7cd90d-ceac-4aa6-8a67-71ffd1b6c746)
![Screenshot 2024-10-29 131948](https://github.com/user-attachments/assets/4dec606e-30bc-44dc-b01d-9cb652224f10)
![Screenshot 2024-10-29 132002](https://github.com/user-attachments/assets/1e67d019-d1a2-41d2-83d0-7ecb5eebe6c6)
![Screenshot 2024-10-29 132013](https://github.com/user-attachments/assets/92210b85-c235-45d3-a5db-9413ea7f02d3)
![Screenshot 2024-10-29 132109](https://github.com/user-attachments/assets/70dd869e-5cc4-41de-9d90-88bfd7183cad)


### Web Server 3

1. Launch another new EC2 instance with RHEL Operating System

 ![Screenshot 2024-10-29 132150](https://github.com/user-attachments/assets/529c9209-a2cb-4255-b155-4ac9cb70db4a)


2. Install NFS Client
    ```bash
    sudo yum install nfs-utils nfs4-acl-tools -y
    ```
![Screenshot 2024-10-29 132201](https://github.com/user-attachments/assets/2d127dac-b2da-4585-a814-ce604bd4d6f7)


3. Mount /var/www/ and target the NFS server's export for apps. NFS Server private IP address = 172.31.1.209
    ```bash
    sudo mkdir /var/www
    sudo mount -t nfs -o rw,nosuid 172.31.1.209:/mnt/apps /var/www
    ```
4. Verify that NFS was mounted successfully by running `df -h`. Ensure that the changes will persist after reboot.

    mount app

    ```bash
    sudo vi /etc/fstab
    ```
    Add the following line
    ```plaintext
    172.31.1.209:/mnt/apps /var/www nfs defaults 0 0
    ```
![Screenshot 2024-10-29 133000](https://github.com/user-attachments/assets/b1fb047b-d8bb-4043-90d7-56fb51e3ce6b)



5. Install Remi's repository, Apache and PHP
    ```bash
    sudo yum install httpd -y
    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    sudo dnf module reset php
    sudo dnf module enable php:remi-8.2
    sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    sudo systemctl status php-fpm
    sudo setsebool -P httpd_execmem 1
    ```
  ![Screenshot 2024-10-29 133022](https://github.com/user-attachments/assets/71f710d1-d1bf-42ee-a819-2b9002fca603)
![Screenshot 2024-10-29 133035](https://github.com/user-attachments/assets/c663a765-1bda-4836-af42-f2012872687d)
![Screenshot 2024-10-29 133049](https://github.com/user-attachments/assets/82c38760-f64a-4b40-9fad-b774d1a41d70)
![Screenshot 2024-10-29 133107](https://github.com/user-attachments/assets/ea47ca93-67b6-464d-9da9-037d85e5d896)

![Screenshot 2024-10-29 133125](https://github.com/user-attachments/assets/99ae6caf-3073-4141-8608-b40c13934ab6)

6. Verify that Apache files and directories are available on the Web Servers in /var/www and also on the NFS Server in /mnt/apps. If the same files are present in both, it means NFS was mounted correctly. `test.txt` file was created from Web Server 1, and it was accessible from Web Server 2.

![Screenshot 2024-10-29 133713](https://github.com/user-attachments/assets/2b9ee9e6-80c8-448e-969f-7b58e3a52d07)


7. Locate the log folder for Apache on the Web Server and mount it to NFS server's export for logs. Repeat step 4 to ensure the mount point persists after reboot.
    ```bash
    sudo vi /etc/fstab
    ```
    Add the following line
    ```plaintext
    172.31.20.140:/mnt/logs /var/log/httpd nfs defaults 0 0
    ```
![Screenshot 2024-10-29 134607](https://github.com/user-attachments/assets/beb3c56f-d8f8-4342-b721-34e55bded858)

![Screenshot 2024-10-29 134552](https://github.com/user-attachments/assets/cea73c31-594e-4e18-9541-7377983c517e)


8. Fork the tooling source code from StegHub GitHub Account
![Screenshot 2024-10-29 135852](https://github.com/user-attachments/assets/8c7cf2dc-156d-43d7-88aa-37ae6317620c)


9. Deploy the tooling Website's code to the Web Server. Ensure that the html folder from the repository is deployed to /var/www/html
![Screenshot 2024-10-29 135927](https://github.com/user-attachments/assets/c9e0b557-08d3-4b6a-88dd-862d0846f1a1)

    Initialize the directory and clone the tooling repository. Ensure to clone the forked repository
    ```bash
    git init
    ```

    Note: Access the website on a browser

    Ensure TCP port 80 is open on the Web Server. If 403 Error occurs, check permissions to the /var/www/html folder and also disable SELinux
    ```bash
    sudo setenforce 0
    ```
    To make the change permanent, open selinux file and set selinux to disable.
    ```bash
    sudo vi /etc/sysconfig/selinux
    ```
    ```plaintext
    SELINUX=disabled
    ```
    ```bash
    sudo systemctl restart httpd
    ```

![Screenshot 2024-10-29 145937](https://github.com/user-attachments/assets/2238beaa-624f-471e-8324-4f8513ffd498)


10. Update the website's configuration to connect to the database (in /var/www/html/function.php file). Apply tooling-db.sql command
     ```bash
     sudo mysql -h <db-private-IP> -u <db-username> -p <db-password < tooling-db.sql
     sudo vi /var/www/html/functions.php
     sudo mysql -h 172.31.8.129 -u webaccess -p tooling < tooling-db.sql
     ```

     Access the database server from Web Server
     ```bash
     sudo mysql -h 172.31.8.129 -u webaccess -p
     ```
 
![Screenshot 2024-10-29 143121](https://github.com/user-attachments/assets/454a451c-1de1-469d-b7dd-b4dfaa273682)
![Screenshot 2024-10-29 143148](https://github.com/user-attachments/assets/bdf83d53-2e12-43e9-94b4-f9ecf788034f)



11. Create in MySQL a new admin user with username: myuser and password: password
     ```sql
     INSERT INTO users(id, username, password, email, user_type, status) VALUES (2, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');
     ```
 ![Screenshot 2024-10-29 143148](https://github.com/user-attachments/assets/45bd4439-a0c8-4ffc-bdf6-34583164118f)


12. Open a browser and access the website using the Web Server public IP address `http://<Web-Server-public-IP-address>/index.php`. Ensure login into the website with myuser user.

     From Web Server 1
     login

     website

     From Web Server 2
     Disable SELinux
     ```bash
     sudo setenforce 0
     ```
     ```plaintext
     SELINUX=disabled
     ```
     disable selinux

     Access the website

![Screenshot 2024-10-29 151158](https://github.com/user-attachments/assets/f4e91b30-76a8-4261-a271-7a394e50936b)

![Screenshot 2024-10-29 152800](https://github.com/user-attachments/assets/7f65112d-f5ce-435c-9321-ef7b6f95ce63)

