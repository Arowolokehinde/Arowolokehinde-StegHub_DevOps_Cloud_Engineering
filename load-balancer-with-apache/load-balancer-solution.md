# Load Balancer Solution With Apache

A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

The diagram below shows the architecture of the solution
![Architecture](./images/architecture-diag.png)

## Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 instance. Make sure that users can be served by Web servers through the Load Balancer.

## Prerequisites

Ensure that the following servers are installed and configured already.

- Two RHEL9 Web Servers
- One MySQL DB Server (based on Ubuntu 24.04)
- One RHEL9 NFS Server

## Prerequisites Configurations

- Apache (httpd) is up and running on both Web Servers.
- `/var/www` directories of both Web Servers are mounted to `/mnt/apps` of the NFS Server.
- All necessary TCP/UDP ports are opened on Web, DB, and NFS Servers.
- Client browsers can access both Web Servers by their Public IP addresses or Public DNS names and can open the `Tooling Website` (e.g, `http://<Public-IP-Address-or-Public-DNS-Name>/index.php`)

# Step 1 - Configure Apache As A Load Balancer

## 1. Create an Ubuntu Server 24.04 EC2 instance and name it Project-8-apache-lb

![ec2 lb](./images/ec2-lb-detail.png)

## 2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group

![Port 80](./images/lb-security-rule.png)

## 3. Install Apache Load Balancer on Project-8-apache-lb and configure it to point traffic coming to LB to both Web Servers.

### i. Install Apache2

- Access the instance

```bash
ssh -i "my-devec2key.pem" ec2-user@18.219.148.178
```
![ssh](./images/ssh-lb.png)

- Update and upgrade Ubuntu

```bash
sudo apt update && sudo apt upgrade
```
![update ubuntu](./images/update-upgrade-lb.png)

- Install Apache

```bash
sudo apt install apache2 -y
```
![Apache](./images/install-apache.png)

```bash
sudo apt-get install libxml2-dev
```
![Apache dependencies](./images/install-dependencies.png)

### ii. Enable the following modules

```bash
sudo a2enmod rewrite

sudo a2enmod proxy

sudo a2enmod proxy_balancer

sudo a2enmod proxy_http

sudo a2enmod headers

sudo a2enmod lbmethod_bytraffic
```
![modules](./images/enable-modules.png)

### iii. Restart Apache2 Service

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```
![Restart apache](./images/restart-apache.png)

## Configure Load Balancing

### i. Open the file 000-default.conf in sites-available

```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```
### ii. Add this configuration into the section `<VirtualHost *:80> </VirtualHost>`

```apache
<Proxy “balancer://mycluster”>
            BalancerMember http://172.31.46.91:80