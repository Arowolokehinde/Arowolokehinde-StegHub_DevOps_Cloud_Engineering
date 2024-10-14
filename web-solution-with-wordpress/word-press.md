# Web Solution With WordPress

## Step 1 - Prepare a Web Server

1. **Launch a RedHat EC2 instance** that serves as a Web Server. Create 3 volumes in the same AZ as the web server EC2, each of 10GB, and attach all 3 volumes one by one to the web server.

2. **Open up the Linux terminal** to begin configuration.
    ```bash
    ssh -i "ec2key.pem" ec2-user@54.174.233.24
    ```

3. **Use `lsblk` to inspect** what block devices are attached to the server.
    ```bash
    lsblk
    ```

4. **Use `df -h` to see all mounts** and free space on the server.
    ```bash
    df -h
    ```

5. **Use `gdisk` utility** to create a single partition on each of the 3 disks.
    ```bash
    sudo gdisk /dev/xvdf
    sudo gdisk /dev/xvdg
    sudo gdisk /dev/xvdh
    ```

6. **Install LVM package**.
    ```bash
    sudo yum install lvm2 -y
    ```

7. **Use `pvcreate` utility** to mark each of the 3 disks as physical volumes (PVs) to be used by LVM.
    ```bash
    sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo pvs
    ```

8. **Use `vgcreate` utility** to add all 3 PVs to a volume group (VG). Name the VG `webdata-vg`.
    ```bash
    sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo vgs
    ```

9. **Use `lvcreate` utility** to create 2 logical volumes, `apps-lv` and `logs-lv`.
    ```bash
    sudo lvcreate -n apps-lv -L 14G webdata-vg
    sudo lvcreate -n logs-lv -L 14G webdata-vg
    sudo lvs
    ```

10. **Verify the entire setup**.
    ```bash
    sudo vgdisplay -v
    lsblk
    ```

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

13. **Use `rsync` utility** to backup all the files in the log directory `/var/log` into `/home/recovery/logs`.
    ```bash
    sudo rsync -av /var/log /home/recovery/logs
    ```

14. **Mount `/var/log` on `logs-lv`** logical volume.
    ```bash
    sudo mount /dev/webdata-vg/logs-lv /var/log
    ```

15. **Restore log files** back into `/var/log` directory.
    ```bash
    sudo rsync -av /home/recovery/logs/log/ /var/log
    ```

16. **Update `/etc/fstab` file** so that the mount configuration will persist after restart of the server.
    ```bash
    sudo blkid
    sudo vi /etc/fstab
    ```

17. **Test the configuration** and reload daemon.
    ```bash
    sudo mount -a
    sudo systemctl daemon-reload
    df -h
    ```

## Step 2 - Prepare the Database Server

1. **Launch a second RedHat EC2 instance** that will serve as the DB Server. Repeat the same steps as for the Web Server, but instead of `apps-lv`, create `db-lv` and mount it to `/db` directory.

2. **Open up the Linux terminal** to begin configuration.
    ```bash
    ssh -i "ec2key.pem" ec2-user@18.209.18.145
    ```

3. **Use `lsblk` to inspect** what block devices are attached to the server.
    ```bash
    lsblk
    ```

4. **Use `gdisk` utility** to create a single partition on each of the 3 disks.
    ```bash
    sudo gdisk /dev/xvdf
    sudo gdisk /dev/xvdg
    sudo gdisk /dev/xvdh
    ```

5. **Install LVM package**.
    ```bash
    sudo yum install lvm2 -y
    ```

6. **Use `pvcreate` utility** to mark each of the 3 disks as physical volumes (PVs) to be used by LVM. Also, use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG `database-vg`.
    ```bash
    sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo vgcreate database-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    sudo vgs
    ```

7. **Use `lvcreate` utility** to create a logical volume, `db-lv`.
    ```bash
    sudo lvcreate -n db-lv -L 20G database-vg
    sudo lvs
    ```

8. **Use `mkfs.ext4` to format** the logical volumes with ext4 filesystem and mount `/db` on `db-lv`.
    ```bash
    sudo mkfs.ext4 /dev/database-vg/db-lv
    sudo mount /dev/database-vg/db-lv /db
    ```

9. **Update `/etc/fstab` file** so that the mount configuration will persist after restart of the server.
    ```bash
    sudo blkid
    sudo vi /etc/fstab
    ```

10. **Test the configuration** and reload daemon.
    ```bash
    sudo mount -a
    sudo systemctl daemon-reload
    df -h
    ```

## Step 3 - Install WordPress on the Web Server EC2

1. **Update the repository**.
    ```bash
    sudo yum -y update
    ```

2. **Install wget, Apache, and its dependencies**.
    ```bash
    sudo yum install wget httpd php-fpm php-json
    ```

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

4. **Configure SELinux Policies**.
    ```bash
    sudo chown -R apache:apache /var/www/html
    sudo chcon -t httpd_sys_rw_content_t /var/www/html -R
    sudo setsebool -P httpd_execmem 1
    sudo setsebool -P httpd_can_network_connect=1
    sudo setsebool -P httpd_can_network_connect_db=1
    sudo systemctl restart httpd
    ```

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

6. **Install MySQL on DB Server EC2**.
    ```bash
    sudo yum update -y
    sudo yum install mysql-server -y
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    sudo systemctl status mysqld
    ```

7. **Configure DB to work with WordPress**.
    ```bash
    sudo mysql_secure_installation
    sudo mysql -u root -p
    CREATE DATABASE wordpress_db;
    CREATE USER 'wordpress'@'172.31.31.27' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
    GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress'@'172.31.31.27' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    show databases;
    exit
    sudo vi /etc/my.cnf
    sudo systemctl restart mysqld
    ```

8. **Configure WordPress to connect to remote database**.
    ```bash
    sudo yum install mysql-server
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    sudo systemctl status mysqld
    cd /var/www/html
    sudo vi wp-config.php
    sudo systemctl restart httpd
    sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
    sudo mysql -h 172.31.30.142 -u wordpress -p
    show databases;
    exit;
    ```

9. **Access the web page** again with the Web Server public IP address and install WordPress on the browser.

At this point, the implementation of this project is complete and WordPress is available to be used.