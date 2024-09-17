# Web Stack Implementation (LAMP Stack) in AWS

## Introduction
The LAMP stack is a widely-used open-source web development platform known for its reliability and flexibility. It comprises four key components:
- **Linux** (the operating system)
- **Apache** (the web server)
- **MySQL** (the database management system)
- **PHP, Perl, or Python** (the programming languages)

Each component plays a crucial role in creating a fully functional web application. This documentation provides detailed guidance on setting up, configuring, and utilizing the LAMP stack, allowing developers to build dynamic websites and applications efficiently. It covers installation procedures, configuration tips, and best practices for optimizing performance and security.

## Step 0: Prerequisites

1. **EC2 Instance**
   - Type: `t2.micro`
   - OS: Ubuntu 24.04 LTS (HVM)
   - Region: us-east-1
   - Launched using the AWS console.
   ![EC2 Instance](https://github.com/user-attachments/assets/9391bb31-dcb2-4a67-9884-001b3afc3081)
   ![EC2 Details](https://github.com/user-attachments/assets/e9702aa4-3799-4ce9-af66-222f5b295ed8)

2. **SSH Key Pair**
   - Created a key pair named `my-ec2-key` for SSH access on port 22.

3. **Security Group Configuration**
   - Inbound Rules:
     - Allow traffic on port 80 (HTTP) from anywhere.
     - Allow traffic on port 443 (HTTPS) from anywhere.
     - Allow traffic on port 22 (SSH) from any IP address (default).
   ![Security Group](https://github.com/user-attachments/assets/a7f792cf-25bf-4fb5-8319-2b2208bf6424)

4. **Networking Configuration**
   - Used the default VPC and Subnet.
   ![VPC and Subnet](https://github.com/user-attachments/assets/8d203e37-44f2-4746-81a3-af373b2dd5ec)

5. **SSH Access**
   - Locate the private key, change permissions, and connect to the instance:
     ```bash
     chmod 400 my-ec2-key.pem
     ssh -i "my-ec2-key.pem" ubuntu@3.84.147.176
     ```
   - Username: `ubuntu`
   - Public IP Address: `3.84.147.176`
   ![SSH Access](https://github.com/user-attachments/assets/ca7788a0-3b4c-49ce-8bd6-e4f777a3fb90)
   ![SSH Connection](https://github.com/user-attachments/assets/04f052fd-9b69-41a2-94eb-2d7978f06595)

## Step 1 - Install Apache and Update the Firewall

1. **Update and Upgrade Packages**
   ```bash
   sudo apt update
   sudo apt upgrade -y
