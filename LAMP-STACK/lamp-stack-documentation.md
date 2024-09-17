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
![Screenshot 2024-09-14 112032](https://github.com/user-attachments/assets/b1290743-6528-486c-a24c-d611b888c561)
