# Implementing a Client-Server Architecture using MySQL Database Management System (DBMS)

## Instructions Followed:

### Step 1: Create and Configure Two Linux-Based Virtual Servers (EC2 Instances in AWS)

#### MySQL Server and MySQL Client
1. Two EC2 Instances of `t2.micro` type and `Ubuntu 24.04 LTS (HVM)` were launched in the `us-east-1` region using the AWS console.
2. The security group inbound rule for both instances was configured with the default SSH on port 22 with source from anywhere.
3. Attached SSH key named `my-ec2-key` to access the instance on port 22.

### Step 2: Install MySQL Server Software on MySQL Server Linux Server
1. Change the private SSH key permission and connect to the instance:
    ```sh
    chmod 400 my-ec2-key.pem
    ssh -i "my-ec2-key.pem" ubuntu@3.87.82.40
    ```
2. Update and upgrade Ubuntu:
    ```sh
    sudo apt update && sudo apt upgrade -y
    ```
3. Install MySQL Server software:
    ```sh
    sudo apt install mysql-server -y
    ```
4. Enable MySQL server:
    ```sh
    sudo systemctl enable mysql
    ```

### Step 3: Install MySQL Client Software on MySQL Client Linux Server
1. Connect to the instance:
    ```sh
    ssh -i "my-ec2-key.pem" ubuntu@54.242.30.171
    ```
2. Update and upgrade Ubuntu:
    ```sh
    sudo apt update && sudo apt upgrade -y
    ```
3. Install MySQL Client software:
    ```sh
    sudo apt install mysql-client -y
    ```

### Step 4: Use MySQL Server's Local IP Address to Connect from MySQL Client
- By default, both EC2 virtual servers are located in the same local virtual network, so they can communicate using local IP addresses.
- MySQL server uses TCP port 3306 by default, so it has to be opened by creating a new entry in inbound rules in MySQL server Security Groups.
- For extra security, access to MySQL server by all IP addresses was not allowed, only the specific local IP address of MySQL client was allowed.

### Step 5: Configure MySQL Server to Allow Connections from Remote Hosts
1. Run the security script of MySQL on MySQL server:
    ```sh
    sudo mysql_secure_installation
    ```
2. Access MySQL shell:
    ```sh
    sudo mysql
    ```
3. Create a user named `client` and a database named `test_db`:
    ```sql
    CREATE USER 'client'@'%' IDENTIFIED WITH mysql_native_password BY 'User123$';
    CREATE DATABASE test_db;
    GRANT ALL ON test_db.* TO 'client'@'%' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    ```
4. Configure MySQL server to allow connections from remote hosts:
    ```sh
    sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
    ```
    - Locate `bind-address = 127.0.0.1`
    - Replace `127.0.0.1` with `0.0.0.0`

### Step 6: Connect Remotely to MySQL Server Database Engine from MySQL Client Linux Server
- Use the MySQL utility to perform this action:
    ```sh
    sudo mysql -u client -h 172.31.18.141 -p
    ```

### Step 7: Verify the Connection and Perform SQL Queries
1. Check that the connection to the remote MySQL server was successful:
    ```sql
    SHOW DATABASES;
    ```
2. Create a table, insert rows into the table, and select from the table:
    ```sql
    CREATE TABLE test_db.test_table (
      item_id INT AUTO_INCREMENT,
      content VARCHAR(255),
      PRIMARY KEY(item_id)
    );

    INSERT INTO test_db.test_table (content) VALUES ("My first choice football club is Chelsea");
    INSERT INTO test_db.test_table (content) VALUES ("My second choice football club is R.Madrid");

    SELECT * FROM test_db.test_table;
    ```

At this point, the project is successfully complete. This deployment is a fully functional MySQL Client-Server setup.