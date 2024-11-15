# Tooling Website deployment automation with Continuous Integration using Jenkins

In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the most popular CI/CD tools.

According to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments which is then automatically built and tested before it is merged with the shared repository.

## Task

Enhance the architecture prepared in the previous project by adding a `Jenkins` server, configure a job to automatically deploy source codes changes from `Git` to `NFS` server.

Here is how the updated architecture looks.
<img width="743" alt="architecture" src="https://github.com/user-attachments/assets/7f0acb97-431e-4f57-9692-5ed6f69bf859">



# Step 1 - Install Jenkins server

## 1. Create an aws EC2 instance based on Ubuntu Server 24.04 LTS and name it `Jenkins`

![Screenshot 2024-11-08 163426](https://github.com/user-attachments/assets/56bee77b-a37e-4186-9007-f041f760d3a6)


## 2. Install JDK since Jenkins is a Java-based application

__Access the instance__

```bash
ssh -i "my-devec2key" ubuntu@3.139.86.58
```
![Screenshot 2024-11-08 163503](https://github.com/user-attachments/assets/32171abf-596c-4521-a6f6-39184fb7e731)



__Update the Instance__

```bash
sudo apt-get update
```
![Screenshot 2024-11-08 163517](https://github.com/user-attachments/assets/bcc9fabf-e5c6-4639-ab2a-5178294b1256)
![Screenshot 2024-11-08 163517](https://github.com/user-attachments/assets/04814239-f663-4e4a-9386-004b07797e5b)



__Download the Jenkins key__

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian/jenkins.io-2023.key
```
![Screenshot 2024-11-08 164118](https://github.com/user-attachments/assets/bdafae5e-9e82-4ba7-a6b3-d0cd4432ddba)


__Add the Jenkins Repository__

```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
```
![Screenshot 2024-11-08 164131](https://github.com/user-attachments/assets/bf22ee2d-5022-46c7-a5b6-bd0798f15677)


__Install Java__

Jenkins requires Java to run, yet not all Linux distributions include Java by default. Additionally, not all Java versions are compatible with Jenkins. Install `OpenJDK 17`

```bash
sudo apt install fontconfig openjdk-17-jre
```
![Screenshot 2024-11-08 164704](https://github.com/user-attachments/assets/45fdfc9a-fe73-40c3-8faf-29651e20c366)



## 3. Install Jenkins

__Update ubuntu__

```bash
sudo apt-get update
```
![Screenshot 2024-11-08 164727](https://github.com/user-attachments/assets/06d510f0-fb5d-46ec-ae75-9a7bc572ba07)


__Install Jenkins__

```bash
sudo apt-get install jenkins -y
```
![Screenshot 2024-11-08 164727](https://github.com/user-attachments/assets/ab490413-fbad-426c-a8f2-b2187cc537fa)


__Ensure jenkins is up and running__

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

![Screenshot 2024-11-08 164738](https://github.com/user-attachments/assets/62fc7b4b-7f51-434d-a13a-cc68d3257e3c)


## 4. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound rule in the EC2 Security Group

![Screenshot 2024-11-08 164901](https://github.com/user-attachments/assets/fe01cc23-726d-435a-ab43-406affa8daec)


## 5. Perform initial Jenkins setup

From a browser access `http://<Jenkins-Server-Public-IP-Address>:8080`
You will be prompted to provide a default admin password.
Retrieve it from the server.

```bash
http://3.83.15.146:8080
```
![Screenshot 2024-11-08 165412](https://github.com/user-attachments/assets/a5a3cbc3-271b-4bd8-a63a-09ebaeefceab)


```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```


Then you will be asked which plugins to install - choose suggested plugins
![Screenshot 2024-11-08 165549](https://github.com/user-attachments/assets/ed006a83-cb9f-4bb2-acc8-ff10b81f730a)



Once plugins installation is done, create an admin user and you will get the jenkins server address.

![Screenshot 2024-11-08 165549](https://github.com/user-attachments/assets/67ff3d93-ecc0-4eb5-9a46-2e654fcd0fa3)


The installation is complete


![Screenshot 2024-11-08 170003](https://github.com/user-attachments/assets/1df88b96-94cb-445e-afd6-ab060bc53d65)

# Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks

In this part, we will learn how to configure a simple Jenkins job/project. This job will be triggered by GitHub webhooks and will execute a build task to retrieve codes from GitHub and store it locally on Jenkins server.

## 1. Enable webhooks in your GitHub repository settings.

On your GitHub repository,

Select Settings > Webhooks > Add webhook


## 2. Go to Jenkins web console, click `New Item` and create a `Freestyle project`

![Screenshot 2024-11-08 171333](https://github.com/user-attachments/assets/27333e6c-889f-4763-b5a9-3acd58d296b0)
![Screenshot 2024-11-08 171425](https://github.com/user-attachments/assets/733e38c6-2e06-4245-9122-f6c47c2fd3f6)


__To connect our GitHub repository, we will need to provide its URL, we can copy from the repository itself__

```bash
https://github.com/Arowolokehinde/tooling.git
```

![Screenshot 2024-11-08 171833](https://github.com/user-attachments/assets/95e9d4d3-d2a3-420a-82bb-a29bdc1bbd04)


In configuration of our Jenkins freestyle project choose Git repository, provide there the link to our Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![Screenshot 2024-11-08 171833](https://github.com/user-attachments/assets/95e9d4d3-d2a3-420a-82bb-a29bdc1bbd04)

Save the configuration and try to run the build. For now we can only do it manually. Click `Build Now` button. After all was configured correctly, the build was successful and was seen under #3
You can open the build and check in Console Output if it has run successfully.

![Screenshot 2024-11-08 171952](https://github.com/user-attachments/assets/929a5f11-ff88-45a6-906c-ede93f9ff275)


But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

## 3. Click Configure our job/project and add these two configurations

__Configure `triggering the job from GitHub webhook` and also Configure `Post-build Actions` to `archive all the files` - files resulted from a build are called artifacts:__

![Screenshot 2024-11-08 173112](https://github.com/user-attachments/assets/f511ac45-ff37-48c5-a590-b81100712e20)


Now, go ahead and make some change in any file in our GitHub repository (e.g. README.MD file) and push the changes to the main branch.



we will see that a new build has been launched automatically by webhook and its results - artifacts, saved on Jenkins server.
![Screenshot 2024-11-08 173758](https://github.com/user-attachments/assets/bcb547b7-08cc-4390-970f-c4cf8d9a9eed)

![Screenshot 2024-11-08 173918](https://github.com/user-attachments/assets/a0167338-7c34-4431-8ebc-561ca9ca90fa)

Now we configured an automated Jenkins job that receives files from GitHub by webhook trigger this method is considered as push because the changes are being pushed and files transfer is initiated by GitHub. There are also other methods: `trigger one job (downstream) from another (upstream)`, `poll GitHub periodically` and others.

By default, the artifacts are stored on Jenkins server locally

```bash
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
![Screenshot 2024-11-08 174225](https://github.com/user-attachments/assets/c44f3916-4923-46de-9470-d309b6a44664)


# Step 3 - Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are more than 1400 plugins available. now we will need a plugin that is called `Publish Over SSH`

### 1. Install Publish Over SSH plugin.

On main dashboard, Select Manage Jenkins > Manage Plugins > Available > Search for Publish over SSH and Install without restart.
![Screenshot 2024-11-08 175342](https://github.com/user-attachments/assets/53fb9edd-b501-41ee-93a2-88ded5ca061a)
![Screenshot 2024-11-08 175603](https://github.com/user-attachments/assets/9d76b3b3-d595-43b9-b6bf-6d281fc9f6aa)


### 2. Configure the job/project to copy artifacts over to NFS server

On main dashboard select `Manage Jenkins > Configure System` menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

- Provide a `private key` (content of .pem file that we use to connect to NFS server via SSH/Putty)
- Arbitrary name
- Hostname - can be `private IP address` of our `NFS` server
- Username - `ec2-user` (since NFS server is based on EC2 with RHEL 9)
- Remote directory - `/mnt/apps` since our Web Servers use it as a mounting point to retrieve files from the NFS server

![Screenshot 2024-11-08 180735](https://github.com/user-attachments/assets/b2128c55-7a85-452e-aebd-dd9a17bcd62b)


Test the configuration and make sure the connection returns Success. N.B that TCP port 22 on NFS server must be open to receive SSH connections

![Screenshot 2024-11-08 180735](https://github.com/user-attachments/assets/0d46f7c7-b813-4b33-8549-de0216359e5d)


Save the configuration, open your Jenkins job/project configuration page and add another one Post-build Action (`Send build artifact over ssh`).

Also, Configure it to send all files produced by the build into our previously defined remote directory In our case we want to copy all files and directories, so we use `**`  If you want to apply some particular pattern to define which files to send - [use this syntax](https://ant.apache.org/manual/dirtasks.html#patterns)

![Screenshot 2024-11-08 180735](https://github.com/user-attachments/assets/223a79ef-5090-4562-9976-5a49ad204e57)


Save this configuration and go ahead, change something in README.MD file in our GitHub Tooling repository

![Screenshot 2024-11-08 181223](https://github.com/user-attachments/assets/fa5fe27f-00c8-4d61-b711-834f188a1df1)


The line created previously in the README.md file have been removed

Webhook will trigger a new job

![Screenshot 2024-11-08 181817](https://github.com/user-attachments/assets/4f7d9d79-1576-4867-8dbf-1289e573803a)


The error in the build #5 above indicates that we need to set permissions for user ec2-user on the NFS server : Ensure the target directory (/mnt) and its contents on the NFS server has the correct permissions. We might need to change ownership or modify the permissions to allow the Jenkins user to write to it.

```bash
sudo chown -R ec2-user:ec2-user /mnt/apps
sudo chmod -R 777 /mnt/apps
```

![Screenshot 2024-11-08 181846](https://github.com/user-attachments/assets/b0709ecb-7b11-42e2-9984-ad978b7f8024)


__Run the build again from jenkins GUI__

Webhook triggers a new job and in the Console Output of the job we get something like this:

```bash
SSH: Transferred 24 file(s)
Finished: SUCCESS
```

![Screenshot 2024-11-08 182037](https://github.com/user-attachments/assets/b39ba506-fd70-4370-b570-c9f395aee097)


__To make sure that the files in `/mnt/apps` have been updated - connect via SSH to our NFS Server and verify README.MD file__

```bash
ls /mnt/apps

or

ls -l /mnt/apps
```
![Screenshot 2024-11-08 182126](https://github.com/user-attachments/assets/2aa1742f-293a-4be7-ad6a-eb35f4c59ce3)


```bash
cat /mnt/apps/README.md
```
![Screenshot 2024-11-08 182208](https://github.com/user-attachments/assets/212220b5-d310-447f-aef9-1d8f650c8c34)


If you see the changes you had previously made in your GitHub - the job works as expected.

Congratulations
