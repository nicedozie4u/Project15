# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY
## INTRODUCTION

This project demostrates how a secure infrastructure inside AWS VPC (Virtual Private Cloud) network is built for a particular company, who uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this. The infrastructure will look like following diagram:

![](./images/proj%2520arch.png)

### Starting Off my AWS Cloud Project

There are few requirements that must be met before i begin:

1. Create an AWS Master account. (Also known as **Root Account**)
2. Within the **Root account**, create a sub-account and name it **DevOps**. (I will need another email address to complete this)
3. Within the Root account, create an AWS **Organization Unit** (OU). Name it **Dev**. (We will launch Dev resources in there)
4. Move the **DevOps** account into the **Dev OU**.
5. Login to the newly created AWS account using the new email address.
6. Create a **free domain** name for my fictitious company at **Freenom** domain registrar [here](https://www.freenom.com/en/index.html?lang=en).

7. Create a **hosted zone** in AWS, and map it to my free domain from Freenom.

### **STEP 1: Setting Up a Sub-account And Creating A Hosted Zone**
---
- Creating a sub-account **'DevOps'** from my AWS master account in the AWS Organisation Unit console

  ![](./images/create%20sub%20acct.png)

- Creating an AWS **Organization Unit** (OU) named **'Dev'** within the **Root account**(I will launched Dev resources  in there) 

  ![](./images/OU%20dev.png)

- Moving the **DevOps account** into the **Dev OU**

  ![](./images/move%20acct%20to%20devOU.png)

  ![](./images/org%20structure.png)

- Logging in to the newly created AWS account with the new email

### **STEP 2: Setting Up a Virtual Private Network (VPC) and Security Group**
---
I'll make reference to the **architectural diagram** and ensure that my configuration is aligned with it.

1. Create a VPC
2. Create subnets as shown in the architecture
3. Create a route table and associate it with public subnets
4. Create a route table and associate it with private subnets
5. Create an Internet Gateway
6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet)
7. Create 3 Elastic IPs
8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)
9. Create a Security Group for: 
   - Nginx Servers
   - Bastion Servers
   - Application Load Balancer
   - Webservers
   - Data Layer     

- Creating a VPC from the VPC console and also edit DNS Hostname and toggle to Enable

  ![](./images/create%20vpc.png)

  ![](./images/dns%20hostname%20disabled.png)

  ![](./images/dns%20hostname%20enable.png)

- Creating subnets for the public and private resources as reference in my architectural diagram. Also checking IP info link: [ipinfo.io/ips](https://ipinfo.io/ips)

  - Public-subnet-1 in my AZ-A(**eu-west-2a**) ipv4 CIDR block **(10.0.1.0/24)**

    ![](./images/pub%20sub%201.png)

  - Public-subnet-2 in my AZ-B (**eu-west-2b**) ipv4 CIDR block **(10.0.3.0/24)**

    ![](./images/pub%20sub%202.png)

  - Private-subnet-1(webserver) in my AZ-A (**eu-west-2a**) ipv4 CIDR block **(10.0.2.0/24)**

    ![](./images/priv%20sub%201.png)

  - Private-subnet-2(webserver) in my AZ-B (**eu-west-2b**) ipv4 CIDR block **(10.0.4.0/24)**

    ![](./images/priv%20sub%202.png)

  - Private-subnet-3(data layer) in my AZ-A (**eu-west-2a**) ipv4 CIDR block **(10.0.5.0/24)**  

    ![](./images/priv%20sub%203.png)

  - Private-subnet-4(data layer) in my AZ-B (**eu-west-2b**) ipv4 CIDR block **(10.0.6.0/24)**   

    ![](./images/priv%20sub%204.png)

  - All Public and Private Subnet in my two AZ has been created  

    ![](./images/all%20subnet.png)

- Creating a **route table** and associating it with **public subnets**  

   ![](./images/pub%20rtb.png)

   ![](./images/pub%20sub%20ass.png)

- Creating another **route table** and associating it with **private subnets** 

   ![](./images/prv%20rtb.png)

   ![](./images/prv%20sub%20ass.png)

- Creating an Internet Gateway and attached it to my VPC  

  ![](./images/igw.png)

  ![](./images/igw%20to%20vpc.png)

- Editing a route in **public route table**, and associating it with the **Internet Gateway**. (This is what allows a public subnet to be accessible from the Internet). 

  Routes tables > public rtb > Action > Edit routes > add route

  ![](./images/edit%20route%20table.png)

- Creating Elastic IPs

  ![](./images/Elastic%20IP.png)

  ![](./images/Elastic%20IP%202.png)

- Creating a **Nat Gateway** and assigning the Elastic IPs to it

  ![](./images/create%20natgateway.png)

- Edit private route table to talk to the natgateway from anywhere

  ![](./images/edit%20prv%20rtb.png)

- Setting up Security Group for **External Application load balancer** in a way that it will allow https traffic from any IP address since inbound traffic will be coming from the internet to my Ext-ALB.  

  ![](./images/sec%20grp%20for%20ALB.png)

- Setting up Security Group for **bastion server** in a way that it will only allow access from the workstations that need to SSH into the bastion servers.

  ![](./images/sec%20grp%20for%20bastion.png)

- Setting up Security Group for **Nginx servers** in a way that it will allow https traffic only from the **Application Load balancer** and allow **bastion servers** to ssh into it. so i will select the ALB and Bastion sec grp respectively as the source.

  ![](./images/sec%20grp%20for%20nginx.png)

- Setting up Security Group for **internal Application Load balancer** in a way that it will allow traffic only from **Nginx servers** so i will select the nginx sec grp as the source

  ![](./images/sec%20grp%20for%20int%20alb.png)

- Setting up Security Group for **Webservers** in a way that it will allow https traffic only from the **internal Application Load balancer** and ssh from bastion server so i will select the internal ALB and Bastion sec grp respectively as the source  

  ![](./images/sec%20grp%20for%20webserver.png)

- Setting Security Group for the **Data Layer** subnet in a way that it will allow TCP port 3306 traffic only from the **webservers.** and **bastion server** so i will select the bastion sec grp and webserver sec grp respectively as the source 

  ![](./images/sec%20grp%20for%20datalayer.png)

### **STEP 3: TLS Certificates From Amazon Certificate Manager (ACM)**
---
I will need **TLS certificates** to handle secured connectivity to my Application Load Balancers (ALB).
 
To create a TLS certificate, i will need to purchased and host a domain name.

- I already  got a free domain **"krisqriz.ga"** from [freenom](https://www.freenom.com/en/index.html?lang=en)

  ![](./images/free%20domain.png)

- Creating a **hosted zone** in the **Route 53** console and mapping it to the **domain name** acquired from **freenom**

  ![](./images/hosted%20zone%201.png)

  ![](./images/hosted%20zone%202.png)

- Navigating to **AWS ACM** to request public certificate.

  ![](./images/AWS%20cert%20man.png)

- Requesting a public wildcard certificate for the **domain name** i registered in **Freenom.** For wildcard domain use **(*.krisqriz.ga)**

  ![](./images/aws%20cert.png)

- Using **DNS** to validate the **domain name** click **certificate ID** > click **create records in Route 53** > click **create record**

  ![](./images/validate%20cert%201.png)

  ![](./images/validate%20cert%202.png)

  ![](./images/validate%20cert%203.png)

  ![](./images/validate%20cert%204.png)

  ![](./images/validate%20cert%205.png)

- Tag the resource  

### **STEP 4: Setting Up EFS Storage for the webservers**
---
- Create an **EFS** filesystem

  ![](./images/create%20efs%201.png)

  ![](./images/create%20efs%202.png)

- Create an **EFS mount target** per **AZ** 1&2 in my VPC **(eu-west 2a and eu-west 2b)** respectively, associate it with both **Private subnets 1** and **private subnet 2** dedicated for **webserver**

- Associate the **Security groups** created earlier for data layer.

  ![](./images/create%20efs%203.png)

  ![](./images/create%20efs%204.png)

- Create two EFS access point 1 for **wordpress** and the other for **tooling**. (Give it a name wordpress & tooling) respectively, POSIX user ID and group ID will be root user ID **"0"** and permission is **"0755"**

  ![](./images/AP%20wordpress.png)

  ![](./images/2%20access%20point.png)

### **STEP 5: Setting Up A Relational Database System**  
---
**Pre-requisite:** Create a **KMS** key from Key Management Service (KMS) to be used to encrypt the database instance.

- **Creating Key Management Service(KMS)**

  ![](./images/kms%201.png)

  ![](./images/kms%202.png)

  ![](./images/kms%203.png)

  ![](./images/kms%204.png)

  ![](./images/kms%205.png)  

- Creating a subnet group and add 2 private subnets (3&4 data Layer)

  ![](./images/subnet%20group.png)

- Create a mysql database

  using the free tier templates. However, with this i wont be able to use the encryption key

   ![](./images/rds%201.png)

   ![](./images/rds%202.png)

   ![](./images/rds%203.png)

### **STEP 6 Creating An AMI Out Of The EC2 Instance For Nginx And Bastion server**
---
Launched **3 EC2 Instance** based on Red Hat of the T2 micro family for **Nginx, bastion** and the one for the two **webservers**

![](./images/launch%203%20instance.png)

![](./images/3%20instance.png)

**For The Bastion Server**
---

After connecting to it through ssh on the terminal, the following commands are run to install some necessary softwares:
  
  `sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

  ![](./images/bastion%20inst%201.png)

  `sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

  ![](./images/bastion%20inst%202.png)

  `sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y`

  ![](./images/bastion%20inst%203.png)
  ```
  systemctl start chronyd 
  
  systemctl enable chronyd
  ```
  ![](./images/bastion%20inst%204.png)

**For Nginx Server**
---
After connecting to EC2 Instance for the Nginx through ssh on the terminal, the following commands are run to install some necessary softwares:

`sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![](./images/nginx%20ins%201.png)

`sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![](./images/nginx%20ins%202.png)

`sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y`
```
systemctl start chronyd 
  
systemctl enable chronyd
```
**Configure Selinux policies**
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```
![](./images/nginx%20ins%203.png)

**Install amazon efs utils for mounting the target on the Elastic file system**
```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```
**setting up self-signed certificate for the nginx instance**
```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
![](./images/nginx%20ins%204.png)

To confirm my cert installation is successful and present in my server: `ls -l /etc/ssl/certs/`

`ls -l /etc/ssl/private/`

![](./images/nginx%20cert.png)

### **STEP 4: Creating An AMI Out Of The EC2 Instance For The Tooling and Wordpress Webservers**

After connecting to the EC2 instance for the tooling and wordpress site through SSH on the terminal, the following commands are run to install some necessary softwares:
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```
![](./images/webserver%20inst%201.png)

![](./images/webserver%20inst%202.png)

**Configure Selinux policies**
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```
![](./images/webserver%20inst%203.png)

**Install amazon efs utils for mounting the target on the Elastic file system**
```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```
**setting up self-signed certificate for the apache webserver instance**
```
yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt

# using vi editor to edit the SSL certificate file path from localhost.crt and localhost.key to ACS.crt and ACS.key respectively

vi /etc/httpd/conf.d/ssl.conf
```
![](./images/webserver%20inst%204.png)

![](./images/webserver%20inst%205.png)

![](./images/webserver%20inst%206.png)

**Creating an AMI out of the webserver EC2 Instance**

select instance > Action > image and templates > create image

![](./images/webserver%20ami.png)

**Creating an AMI out of the Bastion EC2 Instance**

![](./images/bastion%20ami.png)

**Creating an AMI out of the nginx EC2 Instance**

![](./images/nginx%20ami.png)

![](./images/3%20ami.png)

### **STEP 5: Configuring Target Groups**
---
**For Nginx Server**

- Selecting Instances as the target type
- Ensuring the protocol HTTPS on secure TLS port 443
- Ensuring that the health check path is **/healthstatus**

![](./images/nginx%20tg.png)

**For Wordpress Server**

- Selecting Instances as the target type
- Ensuring the protocol HTTPS on secure TLS port 443
- Ensuring that the health check path is **/healthstatus**

![](./images/wordpress%20tg.png)

**For Tooling Server**

- Selecting Instances as the target type
- Ensuring the protocol HTTPS on secure TLS port 443
- Ensuring that the health check path is **/healthstatus**

![](./images/tooling%20tg.png)

![](./images/Tg.png)

### **STEP 6: Configuring Application Load Balancer (ALB)**
---
**For External Load Balancer**

- Selecting Internet facing option
- Ensuring that it listens on **HTTPS** protocol (TCP port 443)
- Ensuring the ALB is created within the appropriate **VPC, AZ** and the right **Subnets**
- Choosing the Certificate already created from **ACM**
- Selecting **Security Group** for the external load balancer
- Selecting **Nginx** Instances as the **target group**


![](./images/ext%20ALB.png)

![](./images/ext%20alb%202.png)

**For Internal Load Balancer**

- Setting the Internal ALB option
- Ensuring that it listens on **HTTPS** protocol (TCP port 443)
- Ensuring the ALB is created within the appropriate **VPC**, **AZ** and **Subnets**
- Choosing the Certificate already created from **ACM**
- Selecting Security Group for the internal load balancer
- Selecting **webserver** Instances as the target group
- Ensuring that health check passes for the target group

![](./images/int%20ALB.png)

![](./images/LB.png)

NOTE: Based on my Internal Load balancer configuration, my default preference, all traffic is forwarded to the webserver. I will now set a rule to catch tooling request and forward to tooling target.

```
Select int-ALB > Listeners > view/edit rules > insert rule > Add condition > Host header > Add action > Forward to > tooling target grp > Save
```

![](./images/conf%20rule%201.png)

![](./images/conf%20rule%202.png)

### **STEP 7: Creating A Launch Template**
---
**For Bastion Server**

- Setting up a launch template with the **Bastion AMI**
- Ensuring the Instances are launched into the **public subnet**
- Entering the Userdata to update yum package repository and install **ansible** and **mysql**

![](./images/bastion%20template.png)

![](./images/bastion%20template%202.png)

![](./images/bastion%20template%203.png)

![](./images/bastion%20template%204.png)

**For Nginx Server**

- Setting up a launch template with the Nginx AMI
- Ensuring the Instances are launched into the public subnet
- Assigning appropriate security group
- Entering the Userdata to update yum package repository and install Nginx

![](./images/nginx%20temp%201.png)

![](./images/nginx%20temp%202.png)

![](./images/nginx%20temp%203.png)

**For Wordpress Server**

- Setting up a launch template with the Webserver AMI
- Ensuring the Instances are launched into the Private subnet
- Assigning appropriate security group
- Configure Userdata to update yum package repository and install wordpress and apache server

![](./images/wordpress%20temp%201.png)

![](./images/wordpress%20tem%202.png)

![](./images/wordpress%20temp%203.png)

**For Tooling Server**

- Setting up a launch template with the Webserver AMI
- Ensuring the Instances are launched into the Private subnet
- Assigning appropriate security group
- Configure Userdata to update yum package repository and install apache server

![](./images/tooling%20temp%201.png)

![](./images/tooling%20temp%202.png)

![](./images/tooling%20temp%203.png)

- All 4 Launch templates created successfully

![](./images/all%20launch%20temp.png)

### **STEP 8: Configuring AutoScaling Group**
---
- Selecting the right launch template
- Selecting the VPC
- Selecting both public subnets 1&2
- Enabling Application Load Balancer for the AutoScalingGroup (ASG)
- Selecting the target group you created before
- Ensuring health checks for both EC2 and ALB
- Setting the desired capacity, Minimum capacity and Maximum capacity to 2
- Setting the scale out option if CPU utilization reaches 90%
- Activating SNS topic to send scaling notifications

**ASG For Bastion Server**

![](./images/bastion%20asg%201.png)

![](./images/bastion%20asg%202.png)

![](./images/bastion%20asg%203.png)

![](./images/bastion%20asg%204.png)

![](./images/bastion%20asg%205.png)

**ASG For Nginx**

![](./images/nginx%20asg%201.png)

![](./images/nginx%20asg%202.png)

![](./images/nginx%20asg%203.png)

![](./images/nginx%20asg%204.png)

![](./images/nginx%20asg%205.png)

**NOTE:** Before i configure my **auto scaling group** for my **webserver**, Using **ssh Agent** on my **bastion server** on **mobaxterm** i will access my **RDS endpoint** and create **wordpress** and **tooling** **database**.

- **Configure MobaXterm**

![](./images/mobaxtern%20conf%20for%20ssh%20agent.png)

- **Using ssh agent goto to bastion server**

`ssh - A ec2-user@3.8.155.64`

![](./images/ssh%20into%20bastion.png)

- Goto mysql using my RDS endpoint as host, user name and password.
- And create wordpress and tooling database

![](./images/rds%20endpoint.png)

![](./images/create%20database.png)

**ASG For Wordpress**

![](./images/wordpress%20asg%201.png)

![](./images/wordpress%20asg%202.png)

![](./images/wordpress%20asg%203.png)

![](./images/wordpress%20asg%204.png)

![](./images/wordpress%20asg%205.png)

** ASG For Tooling Server**
 
 NOTE: same steps for wordpress ASG applies
 - Select tooling template
 - Select my vpc
 - select private subnet 1&2
 - Click Attach an existing Balancer and select tooling LB target group
 - Ensure that ELB health check type is clicked also as EC2 is picked by default.
 - Click target tracking scalling policy and input targer value to 90
 - Add notifiaction and input my SNS topic
 - Add tag. Then proceed to create auto scaling group

 NOTE: I will proceed to delete the instance (nginx, webserver and bastion) i created used to launch templates

### **STEP 9: Creating DNS Records In The Route53 For the Tooling And Wordporess site**
---
![](./images/tooling%20dns%20rec.png)

![](./images/dns%20rec.png)

### **STEP 10: Result**
---
Pasting the url http://wordpress.krisqriz.ga/ and http://www.tooling.krisqriz.ga/ will route traffic from the **External ALB** to the **nginx** will serve as my reverse proxy and then to the server for **wordpress** and **tooling** site respectively through the **internal ALB**

- **Wordpress:**

![](./images/wordpress%20login.png)

![](./images/wordpress%20login%20202.png)

- **Tooling:**

![](./images/tooling%20login.png)

### End of Project....