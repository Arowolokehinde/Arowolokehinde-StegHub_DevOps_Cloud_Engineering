# Web Solution With WordPress

## Step 1 - Prepare a Web Server

1. **Launch a RedHat EC2 instance** that serves as a Web Server. Created 3 volumes in the same AZ as the web server EC2, each of 10GB, and attached all 3 volumes one by one to the web server.
![Screenshot 2024-10-12 221248](https://github.com/user-attachments/assets/7b36c8b4-5c3a-436c-bd3a-b8654d82d061)
![Screenshot 2024-10-12 221421](https://github.com/user-attachments/assets/2b522080-2fa7-45e6-a115-244572108f3a)
![Screenshot 2024-10-12 221919](https://github.com/user-attachments/assets/d6e63566-4f48-4986-b619-7488ef3dac31)


2. **Open up the Linux terminal** to begin configuration.
    ```bash
    ssh -i "ec2key.pem" ec2-user@44.204.0.167
    ```
    ![Screenshot 2024-10-12 223204](https://github.com/user-attachments/assets/32a2a72e-af86-4d42-b3f1-8d6b86b40db5)


3. **Use `lsblk` to inspect** what block devices are attached to the server.
    ```bash
    lsblk
    ```

4. **Use `df -h` to see all mounts** and free space on the server.
    ```bash
    df -h
    ```
![Uploading Screenshot 2024-10-12 223313.png…]()

5. **Use `gdisk` utility** to create a single partition on each of the 3 disks.
    ```bash
    sudo gdisk /dev/xvdf
    ```
![Screenshot 2024-10-12 230218](https://github.com/user-attachments/assets/d69eb329-ae2b-4e8a-9b1e-08e5d3687b6a)

```
    sudo gdisk /dev/xvdg
```
![Screenshot 2024-10-12 230322](https://github.com/user-attachments/assets/b716e399-bcd7-45b9-b564-11b55dc2964e)

```
    sudo gdisk /dev/xvdh
```
![Screenshot 2024-10-12 230355](https://github.com/user-attachments/assets/7b322d91-ac5b-451b-942c-f4dd31cc7b05)

 **use n to add a new partition, p to print partition and w to write to the partition**

5b. Use lsblk utility to view the newly configured partitions on each of the 3 disks
```
    lsblk
```
![Screenshot 2024-10-12 230421](https://github.com/user-attachments/assets/8b402c68-4c38-4cb8-b0e6-50d3a08a6937)


6. **Install LVM package**.
    ```bash
    sudo yum install lvm2 -y
    ```
![Screenshot 2024-10-12 230605](https://github.com/user-attachments/assets/7e442bec-8afc-45c0-abae-f918eba9b4d4)

6b. **use sudo lvmdiskscan to check for available partitions**
```
    sudo lvmdiskscan
```
![Screenshot 2024-10-12 230733](https://github.com/user-attachments/assets/b8b94d65-25c0-416e-86c8-271d8ab230ef)


7. **Use `pvcreate` utility** to mark each of the 3 disks as physical volumes (PVs) to be used by LVM.
    ```bash
    sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo pvs
    ```
![Screenshot 2024-10-12 230903](https://github.com/user-attachments/assets/2143be1f-79c3-4f42-93d4-22870418c530)


8. **Use `vgcreate` utility** to add all 3 PVs to a volume group (VG). Name the VG `webdata-vg`.
    ```bash
    sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo vgs
    ```
![Screenshot 2024-10-12 231245](https://github.com/user-attachments/assets/81bfe361-9460-48d6-bb97-c6224ac24cf6)

9. **Use `lvcreate` utility** to create 2 logical volumes, `apps-lv` and `logs-lv`.
    ```bash
    sudo lvcreate -n apps-lv -L 14G webdata-vg
    sudo lvcreate -n logs-lv -L 14G webdata-vg
    sudo lvs
    ```
![Screenshot 2024-10-12 231518](https://github.com/user-attachments/assets/ad9448cb-2d3c-4e7a-9d1b-9df667d6fcd7)

![Screenshot 2024-10-12 231648](https://github.com/user-attachments/assets/fc31cbf8-03d0-4ae3-a7c0-bca5dcde26b1)

10. **Verify the entire setup**.
    ```bash
    sudo vgdisplay -v
    lsblk
    ```
![Screenshot 2024-10-12 231856](https://github.com/user-attachments/assets/dd48d12e-21d2-4eb5-b8dd-8257ea73d39b)
![Screenshot 2024-10-12 231925](https://github.com/user-attachments/assets/f91de6d8-76d8-4f68-a790-bc2b83a304fe)



11. **Use `mkfs.ext4` to format** the logical volumes with ext4 filesystem.
    ```bash
    sudo mkfs.ext4 /dev/webdata-vg/apps-lv
    sudo mkfs.ext4 /dev/webdata-vg/logs-lv
    ```


12. **Create directories** to store website files and backup log data.
    ```bash
    sudo mkdir -p /var/www/html
    sudo mkdir -p /home/recovery/logs
    sudo mount /dev/webdata-vg/apps-lv /var/www/html
    ```
![Screenshot 2024-10-13 120859](https://github.com/user-attachments/assets/e883ddf9-3833-4395-84d7-801190174395)


13. **Use `rsync` utility** to backup all the files in the log directory `/var/log` into `/home/recovery/logs`.
    ```bash
    sudo rsync -av /var/log /home/recovery/logs
    ```
![Screenshot 2024-10-13 120859](https://github.com/user-attachments/assets/7e6d7c81-9f7c-4459-9192-452c3730235d)


14. **Mount `/var/log` on `logs-lv`** logical volume.
    ```bash
    sudo mount /dev/webdata-vg/logs-lv /var/log
    ```

15. **Restore log files** back into `/var/log` directory.
    ```bash
    sudo rsync -av /home/recovery/logs/log/ /var/log
    ```
![Screenshot 2024-10-13 121109](https://github.com/user-attachments/assets/e00f160f-d955-4b64-9fe0-9dc9bdae1808)

16. **Update `/etc/fstab` file** so that the mount configuration will persist after restart of the server.
    ```bash
    sudo blkid
    sudo vi /etc/fstab
    ```
![Screenshot 2024-10-13 121423](https://github.com/user-attachments/assets/3821e5b1-8c31-4763-b67c-e84b4a9fb44a)
![Screenshot 2024-10-13 122420](https://github.com/user-attachments/assets/273d1c2e-e7b2-487a-a803-5aed3ed90aac)

17. **Test the configuration** and reload daemon.
    ```bash
    sudo mount -a
    sudo systemctl daemon-reload
    df -h
    ```

## Step 2 - Prepare the Database Server

1. **Launch a second RedHat EC2 instance** that will serve as the DB Server. Repeat the same steps as for the Web Server, but instead of `apps-lv`, create `db-lv` and mount it to `/db` directory.
![Screenshot 2024-10-13 123137](https://github.com/user-attachments/assets/dfa5a9a1-f809-4436-8f90-8aaf2a40b04e)
![Screenshot 2024-10-13 123232](https://github.com/user-attachments/assets/1bd17d46-5648-477d-bfa0-09da79c580e8)
![Screenshot 2024-10-13 123644](https://github.com/user-attachments/assets/0bc98d0e-912e-4a17-8f93-e6f4dfab9bea)
![Screenshot 2024-10-13 123701](https://github.com/user-attachments/assets/7fb636e6-9d5d-4c4f-b29e-4eaada604484)



2. **Open up the Linux terminal** to begin configuration.
    ```bash
    ssh -i "ec2key.pem" ec2-user@54.210.146.5
    ```
![Screenshot 2024-10-13 123934](https://github.com/user-attachments/assets/74263ca4-40b6-46a1-8efa-ba8fc8482f34)


3. **Use `lsblk` to inspect** what block devices are attached to the server.
    ```bash
    lsblk
    ```
![Screenshot 2024-10-13 123934](https://github.com/user-attachments/assets/19b8e7bd-e8e8-4778-bfc6-b0dfb4aba545)
![Screenshot 2024-10-13 123954](https://github.com/user-attachments/assets/a7660dd1-23ab-4c5b-b4d2-2869061ed2a7)

4. **Use `gdisk` utility** to create a single partition on each of the 3 disks.
    ```bash
        sudo gdisk /dev/xvdf
    ```
    ![Screenshot 2024-10-13 124046](https://github.com/user-attachments/assets/44f87c9e-796e-499b-bec2-512f07f317f0)

    ```
        sudo gdisk /dev/xvdg
    ```
    ![Screenshot 2024-10-13 124120](https://github.com/user-attachments/assets/42e9cf51-6474-4390-b757-7cbf2a8044f5)

    ```
       sudo gdisk /dev/xvdh
    ```
![Screenshot 2024-10-13 124213](https://github.com/user-attachments/assets/20794da4-02e9-423e-9081-98ad2b2a700f)
![Screenshot 2024-10-13 124234](https://github.com/user-attachments/assets/05472452-ec10-49a8-9783-8e0a4443dc65)

6. **Install LVM package**.
    ```bash
    sudo yum install lvm2 -y
    ```
![Screenshot 2024-10-13 124524](https://github.com/user-attachments/assets/89023f7a-e796-4d38-bbd5-dc813af03db9)
![Screenshot 2024-10-13 124536](https://github.com/user-attachments/assets/3209359e-0b10-495c-b427-cfe9195e37b4)

7. **Use `pvcreate` utility** to mark each of the 3 disks as physical volumes (PVs) to be used by LVM. Also, use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG `database-vg`.
    ```bash
    sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo vgcreate database-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo vgs
    ```
![Screenshot 2024-10-13 124624](https://github.com/user-attachments/assets/b8d0bc55-0da7-47d8-9b78-146a13759d2f)
![Screenshot 2024-10-13 124703](https://github.com/user-attachments/assets/5a6c5fc9-b058-49d4-8e65-3c8b0f7d9454)
![Screenshot 2024-10-13 124930](https://github.com/user-attachments/assets/55ab436c-930e-47a0-99d3-af007d3c512d)

8. **Use `lvcreate` utility** to create a logical volume, `db-lv`.
    ```bash
    sudo lvcreate -n db-lv -L 20G database-vg
    sudo lvs
    ```
![Screenshot 2024-10-13 125240](https://github.com/user-attachments/assets/da84b100-0332-4655-898e-b9647be8c67b)

![Screenshot 2024-10-13 125347](https://github.com/user-attachments/assets/7ef8f46e-4b46-46ba-97ee-47c621a62787)

9. **Use `mkfs.ext4` to format** the logical volumes with ext4 filesystem and mount `/db` on `db-lv`.
    ```bash
    sudo mkfs.ext4 /dev/webdata-vg/db-lv
    sudo mount /dev/webdata-vg/db-lv /db
    ```
![Screenshot 2024-10-13 125933](https://github.com/user-attachments/assets/e3292f51-5c01-451e-8551-394e4618a817)


10. **Update `/etc/fstab` file** so that the mount configuration will persist after restart of the server.
    ```bash
    sudo blkid
    sudo vi /etc/fstab
    ```
![Screenshot 2024-10-13 130059](https://github.com/user-attachments/assets/00a120da-66b1-46cf-9ac6-952a0a75c789)
![Screenshot 2024-10-13 130446](https://github.com/user-attachments/assets/ab986261-b95e-4cf7-91a0-7cf9848f45b9)

11. **Test the configuration** and reload daemon.
    ```bash
    sudo mount -a
    sudo systemctl daemon-reload
    df -h
    ```
![Screenshot 2024-10-13 130529](https://github.com/user-attachments/assets/fe1d3ac1-4c18-43de-a6a2-44e851d5a53d)

## Step 3 - Install WordPress on the Web Server EC2

1. **Update the repository**.
    ```bash
    sudo yum -y update
    ```
![Screenshot 2024-10-13 133207](https://github.com/user-attachments/assets/e620d98c-9a93-4d33-8d6d-93df5515ac8f)

2. **Install wget, Apache, and its dependencies**.
    ```bash
    sudo yum install wget httpd php-fpm php-json
    ```
![Screenshot 2024-10-13 133331](https://github.com/user-attachments/assets/cd59f297-6e90-47cc-b6ca-32f14e7b7bc3)

3. **Install the latest version of PHP** and its dependencies using the Remi repository.
    ```bash
    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    sudo dnf module list php
    sudo dnf module reset php
    sudo dnf module enable php:remi-8.2
    sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
    php -v
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    sudo systemctl status php-fpm
    ```
![Screenshot 2024-10-13 135007](https://github.com/user-attachments/assets/640bc2e3-5bae-4b9d-8ec8-8204d9c0fd62)
![Screenshot 2024-10-13 135229](https://github.com/user-attachments/assets/7a2fc99c-390c-4dcc-90cf-26d78fe1864b)

4. **Configure SELinux Policies**.
    ```bash
    sudo chown -R apache:apache /var/www/html
    sudo chcon -t httpd_sys_rw_content_t /var/www/html -R
    sudo setsebool -P httpd_execmem 1
    sudo setsebool -P httpd_can_network_connect=1
    sudo setsebool -P httpd_can_network_connect_db=1
    sudo systemctl restart httpd
    ```
![Screenshot 2024-10-13 140324](https://github.com/user-attachments/assets/21e49374-091c-4dc4-8b98-baa4a4efef7c)

5. **Download WordPress**.
    ```bash
    sudo mkdir wordpress && cd wordpress
    sudo wget http://wordpress.org/latest.tar.gz
    sudo tar xzvf latest.tar.gz
    cd wordpress/
    sudo cp -R wp-config-sample.php wp-config.php
    cd ..
    sudo cp -R wordpress/. /var/www/html/
    ```
![Screenshot 2024-10-13 140524](https://github.com/user-attachments/assets/55147017-038c-44e3-871b-d92cec6f99bf)
![Screenshot 2024-10-13 140725](https://github.com/user-attachments/assets/25c2f65e-7358-4232-97ae-766fcdad2195)
![Screenshot 2024-10-13 141012](https://github.com/user-attachments/assets/39d4f578-364f-4d83-8d2e-d5cfe94fb634)
![Screenshot 2024-10-13 141217](https://github.com/user-attachments/assets/44425632-5cdf-47d7-adb9-fe6642f039b6)

6. **Install MySQL on DB Server EC2**.
    ```bash
    sudo yum update -y
    sudo yum install mysql-server -y
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    sudo systemctl status mysqld
    ```
![Screenshot 2024-10-13 141413](https://github.com/user-attachments/assets/8f0c7e3d-957a-4972-a846-ce4dfbcb8f5c)

7. **Configure DB to work with WordPress**.
    ```bash
        sudo yum install mysql-server
    ```
    ![Screenshot 2024-10-13 142039](https://github.com/user-attachments/assets/b24a3914-8308-491a-a51e-997242d79b57)
```
    sudo systemctl start mysqld
```
```
    sudo systemctl status mysqld
```
![Screenshot 2024-10-13 142235](https://github.com/user-attachments/assets/1b9fff20-9ae2-4d6c-b390-998e8cf5b808)
```
    sudo mysql
```
```
    CREATE DATABASE wordpress;
    CREATE USER 'kenny'@'172.31.93.154' IDENTIFIED BY '12345678';
    GRANT ALL PRIVILEGES ON wordpress.* TO 'kenny'@'172.31.93.154';
    FLUSH PRIVILEGES;
    SHOW DATABASES;
    exit
```
![Screenshot 2024-10-13 142539](https://github.com/user-attachments/assets/156a8e07-ab29-400f-aaca-bdd2e678ae35)


8. **Configure WordPress to connect to remote database**.
```bash
    sudo yum install mysql-server
```
```
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    sudo systemctl status mysqld
```
```
    cd /var/www/html
```
![Screenshot 2024-10-13 152652](https://github.com/user-attachments/assets/9034a49b-9b91-4242-b4e8-4b4ffa3e43f3)
```
 sudo vi wp-config.php
```
![Screenshot 2024-10-13 152716](https://github.com/user-attachments/assets/52a912ac-f999-445b-be5a-ccd1f1622ad0)
![Screenshot 2024-10-13 152742](https://github.com/user-attachments/assets/8b188ac3-a317-4db2-9317-6ed770bf2f2d)
```
    sudo systemctl restart httpd
    sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
```
![Screenshot 2024-10-13 152811](https://github.com/user-attachments/assets/4b3e0250-a6f3-4a27-a283-b1db0e8c1925)

 ```
     sudo mysql -h 172.31.30.142 -u wordpress -p
    show databases;
    exit;
 ```
![Screenshot 2024-10-13 145137](https://github.com/user-attachments/assets/9fd9f41b-558c-426a-9544-159cc37ca873)


9. **Access the web page** again with the Web Server public IP address and install WordPress on the browser.
![Screenshot 2024-10-13 153146](https://github.com/user-attachments/assets/ed5e8fce-1fdc-42a6-928d-af61fd28c1ac)

![Screenshot 2024-10-13 153210](https://github.com/user-attachments/assets/5baaaba1-841a-4261-b8d5-8d20a0184ace)
At this point, the implementation of this project is complete and WordPress is available to be used.

You can now access your WordPress site by navigating to the public IP address of your Web Server in a web browser. Follow the on-screen instructions to complete the WordPress setup, including setting up your site title, admin username, password, and email.

Congratulations! You have successfully set up a scalable WordPress environment with a dedicated web server and database server. This setup ensures that your WordPress site can handle increased traffic and provides a robust foundation for further customization and development.
