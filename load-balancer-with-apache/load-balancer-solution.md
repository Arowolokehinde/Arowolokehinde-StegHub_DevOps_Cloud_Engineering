# Load Balancer Solution With Apache

A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

The diagram below shows the architecture of the solution


<img width="auto" alt="architecture-diag" src="https://github.com/user-attachments/assets/7f45725d-063f-4ada-b88b-35dafff47852">

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

![Screenshot 2024-11-05 175127](https://github.com/user-attachments/assets/149480e7-2d93-465d-a1f6-9b01bcb78ea4)


## 2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

![Screenshot 2024-11-05 175312](https://github.com/user-attachments/assets/7adef058-1def-4077-9da3-d7589bffdc11)


## 3. Install Apache Load Balancer on Project-8-apache-lb and configure it to point traffic coming to LB to both Web Servers.

### i. Install Apache2

- Access the instance

```bash
ssh -i "my-devec2key.pem" ec2-user@18.219.148.178
```
![Screenshot 2024-11-05 175242](https://github.com/user-attachments/assets/881a2be0-5474-499f-a686-7f04ef5e113a)


- Update and upgrade Ubuntu

```bash
sudo apt update && sudo apt upgrade
```
![Screenshot 2024-11-05 175515](https://github.com/user-attachments/assets/42e09883-daef-4217-810e-f4a48651c28d)


- Install Apache

```bash
sudo apt install apache2 -y
```
![Screenshot 2024-11-05 175531](https://github.com/user-attachments/assets/f3b2f024-ed05-4f02-8e08-6206c5fe09ae)


```bash
sudo apt-get install libxml2-dev
```

![Screenshot 2024-11-05 175545](https://github.com/user-attachments/assets/c090ad67-4b89-4571-8786-2d2ca45dd18b)


### ii. Enable the following modules

```bash
sudo a2enmod rewrite

sudo a2enmod proxy

sudo a2enmod proxy_balancer

sudo a2enmod proxy_http

sudo a2enmod headers

sudo a2enmod lbmethod_bytraffic
```
![Screenshot 2024-11-05 175814](https://github.com/user-attachments/assets/43387f9a-1117-4e9a-b4a4-4e79b00ae7a6)

![Screenshot 2024-11-05 180430](https://github.com/user-attachments/assets/ae8f79f8-fb72-4f75-943d-d4ac4bce7001)

### iii. Restart Apache2 Service

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```
![Screenshot 2024-11-05 180502](https://github.com/user-attachments/assets/08a2ccac-e58e-4f12-ad36-b2cad97c7060)



### iv. Configure Load Balancing

- Open the file `000-default.conf` in `sites-available`

```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```

- Add this configuration into the section `<VirtualHost *:80> </VirtualHost>`

```apache
<Proxy "balancer://mycluster">
    BalancerMember http://172.31.46.91:80 loadfactor=5 timeout=1
    BalancerMember http://172.31.43.221:80 loadfactor=5 timeout=1
    ProxySet lbmethod=bytraffic
    # ProxySet lbmethod=byrequests
</Proxy>

ProxyPreserveHost on
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```
![Screenshot 2024-11-05 181138](https://github.com/user-attachments/assets/ac87a0e5-ad1e-4149-90b6-789145658694)

- Restart Apache

```bash
sudo systemctl restart apache2
```
![Screenshot 2024-11-05 181940](https://github.com/user-attachments/assets/9bfd807a-31c2-41b6-8317-4110c5f34eb7)


The `bytraffic` balancing method will distribute incoming load between the Web Servers according to current traffic load. The proportion in which traffic must be distributed can be controlled by the `loadfactor` parameter.

Other methods such as `bybusyness`, `byrequests`, `heartbeat` can also be adopted.

### 4. Verify that the configuration works

- Access the website using the LB's Public IP address or the Public DNS name from a browser.
![Screenshot 2024-11-05 182102](https://github.com/user-attachments/assets/6b3f81c7-3aed-4084-afd1-fc93389430ec)

![Screenshot 2024-11-05 182042](https://github.com/user-attachments/assets/5ef809d0-4761-4256-b0c1-8f396ebac466)

Note: If in the previous project, `/var/log/httpd` was mounted from the Web Server to the NFS Server, unmount them and ensure that each Web Server has its own log directory.

- Unmount the NFS directory

Check if the Web Server's log directory is mounted to NFS

```bash
df -h
sudo umount -f /var/log/httpd
```

If the directory is busy, the services using it need to be stopped first.

```bash
sudo systemctl stop httpd
```

Check that the directory is unmounted

```bash
df -h
```

- Open two SSH consoles for both Web Servers and run the command:

```bash
sudo tail -f /var/log/httpd/access_log
```
***Web Server 1***

![Screenshot 2024-11-05 182827](https://github.com/user-attachments/assets/82461d5c-9d40-486b-9116-7ee5179538f9)

***Web Server 2***
![Screenshot 2024-11-05 182849](https://github.com/user-attachments/assets/9f584b32-d0b6-4e5f-bf9e-b5dc0592b769)

- Refresh the browser page several times and ensure both Web Servers receive HTTP and GET requests. New records must appear in each web server log file. The number of requests to each server will be approximately the same since `loadfactor` is set to the same value for both servers. This means that traffic will be evenly distributed between them.
![Screenshot 2024-11-05 182827](https://github.com/user-attachments/assets/82461d5c-9d40-486b-9116-7ee5179538f9)

![Screenshot 2024-11-05 182849](https://github.com/user-attachments/assets/9f584b32-d0b6-4e5f-bf9e-b5dc0592b769)

### Optional Step - Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, especially if there are lots of servers to manage. It is best to configure local domain name resolution. The easiest way is to use the `/etc/hosts` file, although this approach is not very scalable, it is very easy to configure and shows the concept well.

- Configure the IP address to domain name mapping for our Load Balancer.

Open the hosts file

```bash
sudo vi /etc/hosts
```

Add two records into the file with Local IP address and arbitrary name for the Web Servers
![Screenshot 2024-11-05 183310](https://github.com/user-attachments/assets/5b54cf3f-578c-407f-979e-c4b33b870728)


- Update the LB config file with those arbitrary names instead of IP addresses

```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```

```apache
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```

![Screenshot 2024-11-05 183458](https://github.com/user-attachments/assets/086b5c1c-727b-4321-b912-85368fc856b2)


- Try to curl the Web Servers from LB locally

```bash
curl http://Web1
curl http://Web2
```
![Screenshot 2024-11-05 183605](https://github.com/user-attachments/assets/6e878393-9105-4085-a27a-58e98270b113)

![Screenshot 2024-11-05 183621](https://github.com/user-attachments/assets/8e31ec9c-8abf-40c6-b440-0c3d8ac9cda9)


Remember, this is only internal configuration and also local to the LB server, these names will neither be 'resolvable' from other servers internally nor from the Internet.

### Conclusion

The `mod_proxy_balancer` module in Apache HTTP Server offers robust features for load balancing, including support for sticky sessions, health checks, and various load balancing algorithms. Properly configuring these options ensures high availability, scalability, and reliability for web applications.
