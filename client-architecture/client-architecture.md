# Implementing a Client-Server Architecture using MySQL Database Management System (DBMS)

## Instructions Followed:

### Step 1: Create and Configure Two Linux-Based Virtual Servers (EC2 Instances in AWS)

#### MySQL Server and MySQL Client
1. Two EC2 Instances of `t2.micro` type and `Ubuntu 24.04 LTS (HVM)` were launched in the `us-east-1` region using the AWS console.
![Screenshot 2024-10-04 164349](https://github.com/user-attachments/assets/fda9c720-b5f2-41ea-92e6-4a1817465980)
![Screenshot 2024-10-04 164511](https://github.com/user-attachments/assets/b2f12845-d5da-45d2-8e45-db55067cb242)

![Screenshot 2024-10-04 165036](https://github.com/user-attachments/assets/ba581084-25cb-400f-b535-fef938506583)

2. The security group inbound rule for both instances was configured with the default SSH on port 22 with source from anywhere.
![Screenshot 2024-10-04 165111](https://github.com/user-attachments/assets/8b3f2b0e-803d-49c2-91ec-e8fd9894d2bf)


3. Attached SSH key named `my-ec2-key` to access the instance on port 22.

### Step 2: Install MySQL Server Software on MySQL Server Linux Server
1. Change the private SSH key permission and connect to the instance:
    ```sh
    chmod 400 my-ec2-key.pem
    ssh -i "my-ec2-key.pem" ubuntu@3.87.82.40
    ```

![Screenshot 2024-10-04 165151](https://github.com/user-attachments/assets/b04f8a60-2453-4de0-9ac0-209b47433648)

2. Update and upgrade Ubuntu:
    ```sh
    sudo apt update && sudo apt upgrade -y
    ```
![Screenshot 2024-10-04 165438](https://github.com/user-attachments/assets/51e252f2-b561-4865-83d7-0b77242f0dab)

3. Install MySQL Server software:
    ```sh
    sudo apt install mysql-server -y
    ```
![Screenshot 2024-10-04 165619](https://github.com/user-attachments/assets/4be362d3-7d42-491b-8d6d-0b314d74819d)

4. Enable MySQL server:
    ```sh
    sudo systemctl enable mysql
    ```
![Screenshot 2024-10-04 165721](https://github.com/user-attachments/assets/78ba873e-cda2-4ca2-9fed-7d3cf825d05c)

### Step 3: Install MySQL Client Software on MySQL Client Linux Server
1. Connect to the instance:
    ```sh
    ssh -i "my-ec2-key.pem" ubuntu@ip
    ```
2. Update and upgrade Ubuntu:
    ```sh
    sudo apt update && sudo apt upgrade -y
    ```
![Screenshot 2024-10-04 170239](https://github.com/user-attachments/assets/df1c71ca-51b7-4d37-b62d-3e195d8102b3)

3. Install MySQL Client software:
    ```sh
    sudo apt install mysql-client -y
    ```

![Screenshot 2024-10-04 170337](https://github.com/user-attachments/assets/e3d73a44-8dea-4b59-b896-ff2d2bf494f1)


### Step 4: Use MySQL Server's Local IP Address to Connect from MySQL Client
- By default, both EC2 virtual servers are located in the same local virtual network, so they can communicate using local IP addresses.
- MySQL server uses TCP port 3306 by default, so it has to be opened by creating a new entry in inbound rules in MySQL server Security Groups.
- For extra security, access to MySQL server by all IP addresses was not allowed, only the specific local IP address of MySQL client was allowed.
![Screenshot 2024-10-04 170659](https://github.com/user-attachments/assets/7f907426-7fbf-49ae-a71c-a07beb39ca4b)

![Screenshot 2024-10-04 170741](https://github.com/user-attachments/assets/f04abd91-317e-44e7-b2f1-3d6f392ba9e7)

### Step 5: Configure MySQL Server to Allow Connections from Remote Hosts
1. Run the security script of MySQL on MySQL server:
    ```sh
    sudo mysql_secure_installation
    ```
![Screenshot 2024-10-04 172859](https://github.com/user-attachments/assets/3bb5ce4d-805d-42a0-9ecd-7f1ac6940ba8)

2. Access MySQL shell:
    ```sh
    sudo mysql
    ```
![Screenshot 2024-10-04 173232](https://github.com/user-attachments/assets/fb455041-4d60-4dfc-a3a0-27f5468522b9)


3. Create a user named `client` and a database named `test_db`:
    ```sql
    CREATE USER 'client'@'%' IDENTIFIED WITH mysql_native_password BY 'User123$';
    CREATE DATABASE test_db;
    GRANT ALL ON test_db.* TO 'client'@'%' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    ```
![Screenshot 2024-10-04 173232](https://github.com/user-attachments/assets/b7e40271-1bcb-4249-b169-ba5b594e27a2)


4. Configure MySQL server to allow connections from remote hosts:
    ```sh
    sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
    ```
    - Locate `bind-address = 127.0.0.1`
    - Replace `127.0.0.1` with `0.0.0.0`

![Screenshot 2024-10-04 171533](https://github.com/user-attachments/assets/91684186-b87e-47b7-a26c-f69dfa87cfa2)

### Step 6: Connect Remotely to MySQL Server Database Engine from MySQL Client Linux Server
- Use the MySQL utility to perform this action:
    ```sh
    sudo mysql -u client -h 172.31.36.132 -p
    ```
![Screenshot 2024-10-04 175541](https://github.com/user-attachments/assets/10abb9e1-7ad7-4544-9327-3c96078fb4f4)


### Step 7: Verify the Connection and Perform SQL Queries
1. Check that the connection to the remote MySQL server was successful:
    ```sql
    SHOW DATABASES;
    ```
![Screenshot 2024-10-04 175744](https://github.com/user-attachments/assets/09b6209e-aba5-4e48-9b7a-4b7e50e4163c)

At this point, the project is successfully complete. This deployment is a fully functional MySQL Client-Server setup.
