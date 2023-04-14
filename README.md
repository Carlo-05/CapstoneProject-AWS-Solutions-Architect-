# Recreate CapstoneProject-AWS-Solutions-Architect- 


## Project Overview

This project provides you with an opportunity to demonstrate the solution design skill that you develop throughout AWS Academy Cloud Architecting course. Your assignment is to design and deploy solution for the following case.

By the end of this project, you should be able to apply the architectural design principles that you learned in this scenario:

- Create Admin group and user.
- Create VPC environment.
- Deploy a PHP application that runs on an Amazon EC2 Instance.
- Create database instance that the PHP application can query.
- Create a MySQL database from a SQL dump file.
- Update application parameter in an AWS System Manager Parameter Store.
- Secure the application to prevent public access to backend system.

Solution Requirements

- Provide secure hosting of MySQL database.
- Provide secure access for administrative user.
- Provide anonymous access to web users.
- Run the website on t2.micro EC2 instance, and provide SSH access to administrator.
- Provide high availability to the website through a load balancer
- Store database connection information in the AWS System Manager Parameter Store

## The Project should look like this.
![Capstone Project](https://github.com/Carlo-05/hello-world/blob/main/CapstoneProjectFinal.png?raw=true)


## Step 1: Create Admin group and user

- Login into your AWS Management Console as an Admin user.
- If you are using a Root Account, kindly create an Admin user.
  - Login into your Root Account, search for IAM in search bar then click.
  - Click on the user groups in the left side of your screen under Access Management.

Note: It is easier to manage user if you create and attached policies on a IAM group.

  - Click on create group, input a group name. In this case I name the group Administrator.
  - Attached a AdministratorAccess policy to the group, then click create.
  - After you finish creating your Administrator group, click users on the left side of your screen under Access Management.
  - Click on add user, provide any username you want.
  - Check box beside **Provide user access to the AWS Management Console.**
  - And select **I want to create an IAM user**
  - Select custom password, and input your desired password. In this project let uncheck the box beside **Users must create a new password at next sign-in.**
  - Click next, in the permission options select **add user to a a group.** Select the Administrator group on the list below.
  - Click next, review your choices. After you finish reviewing you can click create user.
  - Take note of the console sign-in URL.
  - Login into your Admin user.

## Step 2: Create VPC Environment

- Create VPC. Take note of the Region where you created your VPC. In my case I use N.Virginia (us-east-1)
- VPC settings:
  - Resource to create: VPC only
  - Name: **MyVPC**
  - IPv4 CIDR: **10.0.0.0/16**
  - Leave everything in default.
- Create subnet:
  - VPC ID: _select_ MyVPC
  - Subnet settings:
    - Subnet 1 of 4
    - Subnet name: **Public 1**
    - Availability Zone: **us-east-1a**
    - IPv4 CIDR block: **10.0.0.0/24**
  - Add new subnet:
    - Subnet 2 of 4
    - Subnet name: **Public 2**
    - Availability Zone: **us-east-1b**
    - IPv4 CIDR block: **10.0.1.0/24**

    - Subnet 3 of 4
    - Subnet name: **Private 1**
    - Availability Zone: **us-east-1a**
    - IPv4 CIDR block: **10.0.2.0/24**

    - Subnet 4 of 4
    - Subnet name: **Private 2**
    - Availability Zone: **us-east-1b**
    - IPv4 CIDR block: **10.0.3.0/24**
- Create internet gateway
  - Name tag: **MyIGW**
- Click Create internet gateway.
- Attach the created internet gateway into your MyVPC.
- Make sure that the Auto-assign public IPv4 address is enable in your public subnets. To do this click the subnets under the Virtual private cloud on the left side of your screen.
- Select Public 1 on the list. Click actions and select edit subnet settings. Check the box beside Enable auto-assign public IPv4 address then save.
- Do the same to Public 2 subnet.
- Create route table for your public subnets:
  - Name: **Public**
  - VPC: **MyVPC**
- Click Create route table.
- On the Routes tab click edit routes. Click add route:
  - Destination: **0.0.0.0/0**
  - Target: Internet Gateway
    - _Select_ your **MyIGW**
- Save changes.
- On the Subnet associations tab. Click Edit subnet association and select the 2 public subnet on the list.
- Save associations
- Create route table for your private subnets:
  - Name: **Private**
  - VPC: **MyVPC**
- Click Create route table.
- On the Subnet associations tab. Click Edit subnet association and select the 2 private subnet on the list.
- Save associations

## Step 3: Create AWS RDS MySQL

- First, we need to create DB Subnet groups:
  - Name: **MyDBSubnetGroups**
  - Description: **MyDBSubnetGroups**
  - VPC: **MyVPC**
  - Availability Zone: **us-east-1a, us-east-1b**
  - Subnets: **Private 1, Private 2**
- Click Create
- Create database:
  - Database creation method: **Standard create**
  - Engine option: **MySQL**
  - Engine Version: **MySQL 8.0.28**
  - Templates: **Dev/Test**
    - Note: If you are using AWS free tier account. Use the Free tier template, but you

You will not be able to use the Multi-AZ Deployment option. If you have budget then you can use the production and dev/test template

  - Deployment options: **Multi-AZ DB instance**
  - DB instance identifier: **example**
  - Master username: **admin**
  - Master password: **\<input your desired password\>**
  - Confirm Master password: **\<input your desired password\>**
  - VPC: **MyVPC**
  - DB subnet group: **mydbsubnetgroups**
  - VPC security group: _select_ Create new
    - Security group name: **DBSG**
    - Availability Zone: **us-east-1a**
- Additional configuration:
  - Initial database name: **MyDB**
- Leave other options in their default state.
- Create database
- Take note of your DB Endpoint:
  - **example.a\*\*d\*fghij\*.us-east-1.rds.amazonaws.com**

## Step 4: Create IDE environment using AWS Cloud9:

- Name: **MyCapstoneIDE**
- VPC: **MyVPC**
- Subnet: **Public 2**
- Leave other option in default
- Launch your IDE and input this commands
  - **sudo yum install update -y**
  - **sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2**
  - **sudo yum install -y httpd mariadb-server**
  - **sudo systemctl start httpd**
  - **sudo systemctl enable httpd**
  - **sudo wget https://github.com/Carlo-05/CapstoneProject-AWS-Solutions-Architect-/archive/refs/heads/main.zip**
  - **sudo unzip main.zip**
  - **cd CapstoneProject-AWS-Solutions-Architect—main/**
  - **sudo unzip Example.zip**
  - **sudo cp \* /var/www/html**
  - **cd /var/www/html**
  - **sudo rm Example.zip**
- Go to EC2 and check the cloud9 instance security group, add inbound rule:
  - Type: **HTTP**
  - Port: **80**
  - Source: **0.0.0.0/0**
- Get the public IP of the cloud9 instance and paste it in your browser to check if the web app is working.

## Step 5: Import sql file into your Amazon RDS

- First, we configure the inbound rules of Amazon RDS security group: **DBSG**
  - Type: **MYSQL/Aurora**
  - Port: **3306**
  - Source: **Cloud9 security group**
- Verify connection to your Amazon RDS
  - **mysql -u admin -p –host example.a\*\*d\*fghij\*.us-east-1.rds.amazonaws.com**
- Enter your DB password
- After you verify that you can connect to your Amazon RDS, check if there is a database named MyDB that we created earlier:
  - show databases;
    - After you verify that the database exists, exit mysql
  - Exit
- Import the sql file Countrydatadump.sql you downloaded earlier:
  - **Mysql -u admin -p MyDB –host example.a\*\*d\*fghij\*.us-east-1.rds.amazonaws.com \< Countrydatadump.sql**
- Verify if you successfully import the file
  - **mysql -u admin -p –host example.a\*\*d\*fghij\*.us-east-1.rds.amazonaws.com**
  - **show databases;**
  - **use MyDB;**
  - **show tables;**
  - **select \* from countrydata\_final;**
  - **exit**
- Notice if you try to query in the web app, there is no result. Let's connect the Cloud9 instance to the AWS RDS using AWS System Manager to automatically connect the Cloud9 instance to the AWS RDS.

## Step 6: Use AWS System Manager to store database connection information in parameter store.

- Create parameters in parameter store in AWS System Manager
  - Name: **/example/username**
  - Value: **admin**

  - Name: **/example/password**
  - Value: **\<Your DB password\>**

  - Name: **/example/endpoint**
  - Value: **example.a\*\*d\*fghij\*.us-east-1.rds.amazonaws.com**

  - Name: **/example/database**
  - Value: **MyDB**

## Step 7: Create IAM Role. We need this to a give permission to your EC2 to link into your Amazon RDS.

- Create IAM Role:
  - Trusted entity: _select_ **AWS service**
  - Use case: _select_ **EC2**
- Attach a permission policy for your IAM role
  - Type AmazonSSMFullAccess in search bar.
  - _Select_ **AmazonSSMFullAccess**
  - Role name: **Inventory-App-Role**
  - Create Role

## Step 8: Attached the Inventory-App-Role into your Cloud9 instance

- Select Cloud9 instance in your EC2 page
- Select Actions
- Select Security
- Select modify IAM role
- Select Inventory-App-Role
- Update IAM role

Go back to the web browser where you open you web application. Try to query if the web app gives you data.

## Step 9: Create AMI image of the Cloud9 instance.

- Select Cloud9 instance in your EC2 page.
- Select Actions
- Select Image and Templates
- Create image
- Image name: **CapstoneImage**
- Leave everything in their default state.

## Step 10: Create launch template.

- Launch template name: **MyCapstoneTemplate**
- Template version description: **MyCapstoneTemplate**
- AMI: **CapstoneImage**
- Instance type: **t2.micro**
- Key pair name: _select_ Create new key pair
  - Key pair name: **KeyPair**
  - Private key file format: _select_ **.ppk**
  - Create key pair
    - Note: After you create your key pair, it will automatically download your keypair

in your device.

- Network settings:
  - Firewall (Security groups): _select_ Create security group
  - Security group name: **WebAppSG**
  - Description: **WebAppSG**
  - VPC: **MyVPC**
- Advanced details:
  - IAM instance profile: **Inventory-App-Role**
- Create launch template

## Step 11: Create Load Balancer

- Load balancer types: _select_ Application Load Balancer
- Load balancer name: **InternetFacingALB**
- VPC: **MyVPC**
- Mappings: _Select_ us-east-1a and us-east-1b
  - us-east-1a
    - Subnet: _select_ **Public 1**
  - us-east-1b
    - Subnet: _select_ **Public 2**
- Security groups:
  - Create new security group
    - Security group name: **ALBSG**
    - Description: **ALBSG**
    - VPC: **MyVPC**
    - Inbound rules:
      - Type: **HTTP**
      - Source: **0.0.0.0/0**
  - Back to your load balancer page
    - Security groups: _select_ **ALBSG**

Note: Deselect the default Security Group.

- Listeners and routing:
  - Create target group
    - Choose a target type: _select_ Instances
    - Target group name: **WebAppTargetGroup**
    - Next
    - Create target group
  - Protocol: **HTTP**
  - Port: **80**
  - Default action: _Forward to_ **WebAppTargetGroup**
- Crete load balancer
- Take note of the DNS name of your load balancer

## Step 12: Create Auto Scaling Groups

- Name: Auto Scaling group name: **WebAppAutoScale**
- Launch template: **MyCapstoneTemplate**
- Next
- Network:
  - VPC: **MyVPC**
  - Availability Zones and subnets: _select_ **Public 1** and **Public 2**
- Next
- Load balancing: _select_ Attached to an existing load balancer
- Existing load balancers target groups: _select_ **WebAppTargetGroup**
- Next
- Group size:
  - Desired capacity: **2**
  - Minimum capacity: **2**
  - Maximum capacity: **4**
- Next
- Create Auto Scaling group

## Step 13: Create a new security group for your Bastion Host and edit existing security groups

- Create security group:
  - Security group name: **BastionHostSG**
  - Description: **allow ssh access into your web apps**
  - VPC: **MyVPC**
  - Add Inbound Rule:
    - Type: **SSH**
    - Source: _Select_ **My IP**
- Edit WebAppSG security group:
  - Add Inbound rules:
    - Type: **SSH**
    - Source: **BastionHostSG**
    - Save
  - Add Inbound rules:
    - Type: **HTTP**
    - Source: **ALBSG**
    - Save
- Edit DBSG security group:
  - Add Inbound rules:
    - Type: **MYSQL/Aurora**
    - Source: **WebAppSG**
    - Save
- Edit Cloud9 security group:
  - Edit the existing Inbound rule:
    - Change the source from 0.0.0.0/0 to **My IP**
      - Note: The reason for this is to close the connection to the public, and for you to edit and test additional feature of your web app in the future so you can create another AMI image. Because of this, you can update the launch configuration for your auto scaling group. Or the best option is to delete the inbound rule and add a new one when you need to view what your web app look like if you open it in the internet.

## Step 14: Create Bastion Host instance

- Name: **BastionHost**
- AMI: **Amazon Linux 2 AMI (HVM) – Kernel 5.10, SSD Volume Type**
- Instance type: **t2.micro**
- Keypair: _select_ **KeyPair**
- Network Settings:
  - VPC: **MyVPC**
  - Subnet: **Public 1**
  - Firewall (security group): Select existing security group
    - Common security groups: **BastionHostSG**
- Launch instance

## Step 15: Test SSH connection to your web application instance.

- Download Putty and Putty Agent
  - Open Putty Agent:
    - Add key: _select_ **KeyPair**
      - Note: This is the key pair we created earlier in step 10.
  - Open Putty:
    - Copy the public IP of your Bastion Host
    - In the session category, paste your Bastion Host public IP address in Host Name.
    - In the Connection category input a value of 30 in Seconds between keepalives (0 to turn off)
    - Under the Connection category expand SSH, and select the Auth
    - Check or enable the Allow agent forwarding.
    - Open and accept connection.
  - In the Putty terminal
    - Login as: **ec2-user**
  - Test the connection to your web app
    - Copy the private IPs of the web app instances
    - To connect, use this command:
      - **ssh ec2-user@\<private ip of web app instance\>**
        - Accept connection
    - If the connection is successful type the command "exit" to go back to the Bastion Host. Try to connect to another web app instance.

## Step 16: Test your Capstone Project

- Copy the DNS name of your Load balancer you take note earlier and paste in in your web browser. It should be working now. Try the query function of your app.
