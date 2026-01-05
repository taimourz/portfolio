---
title:  Deploy your web application on AWS EC2
date: 2026-1-4
tags: aws
---

## Contents
1. [Introduction](#1-introduction)
    - a: Create account
    - b: Regions
    - c: Availability Zones
2. [EC2](#2-ec2)
3. [Security Group](#3-security-group)
    - inbound rules
    - outbound rules
4. [Deploy your webapp](#4-deploy-your-webapp)
5. [Destroy the EC2 instance](#5-destroy-the-ec2-instance)


## 1. Introduction

Suppose you have developed an app which listens on Port 8080. Now you want the showcase it to the world. For this, we have to do the following steps:

1. Purchase a public static IP address. So that, another person can visit `"http://<our_IP>:5173" ` to access your site.
2. Keep it running 24/7
3. Manage resources (RAM, CPU, etc) in case users increase 
4. Manage Security (cyber-attacks), protect from physical damage (wear and tear etc) 

Instead of doing all of this, we can just use different cloud providers like AWS Azure etc

### 1a. Create Account

Creating an [AWS account](https://aws.amazon.com) is as simple as creating an account on a social media app. Its free tier is 12 months which is perfect for learning


### 1b. Regions

An AWS Region is a geographical area where AWS has multiple data centres. Each Region is a completely independent environment. 

> - We need ot be careful in which region we create our resources because the resources of one region are not visible (or available) in another region.
> - Always select the region closest to your target users for better performance.
> - Each region is associated with a unique identifier name. for eg Mumbai, it is "ap-south-1".

### 1c. Availability Zones

Availability Zones are isolated locations within a Region. Each AWS Region has multiple Availability Zones, typically three or more.

**Example**: In the Mumbai region (ap-south-1), AWS can have its physical data centres kept in different locations of Mumbai within a 100 km radius. These locations inside Mumbai are called Availability Zones.

> Availability Zones provide redundancy and ensure high availability. By deploying our resources across multiple AZs, we protect our applications from failure in a single data centre.

Mumbai has 3 data centres currently:
- ap-south-1a
- ap-south-1b
- ap-south-1c

"a", "b", "c", … suffix is attached after the region code for different availability zones.


## 2. EC2

The machine that AWS provides which has a public IP address and is kept in a cloud is called EC2 (Elastic Compute Cloud). It is a virtual computer which we take on rent that can be used to run applications, store data, and more.

> 

> **Instance**: The virtual machine (EC2) that we launch is known as an Instance. Bascially, It's a virtual server in AWS. It runs an operating system (like Linux or Windows) and can be configured with different amounts of CPU, memory, and storage. 
>
> **Instance Type**: This defines the hardware specifications of the instance, like the number of CPUs, amount of RAM, etc. Examples include "t2.micro" (a small, general-purpose instance) and "m5.large" (a more powerful instance). 
>
> **Elastic IP Address**: A static, public IP address that you can associate with your instance, allowing it to be reachable from the internet.

## 3. Security Group

A Security Group is a set of rules that control the traffic allowed to and from our EC2 instances. Just like a firewall that specifies which traffic is allowed based on IP address, port, and protocol.

### 3a. Inbound Rules
Define which traffic is allowed to come from outside world to our EC2. 

>**Example**:
> - We may set SSH (Port 22) traffic only from our IP address. Now, only we can SSH into your EC2 machine terminal.
> - We can expose Port 8080 to allow from anywhere so that anyone in the world can access our app which is running on Port 8080.

### 3b. Outbound Rules
Define which traffic is allowed to go outside our EC2 instance to the world. 

>**Example**:
> - Suppose our database instance is hosted somewhere else then we only allow our EC2 IP address to connect to your database. So, no one else backend can make the connection to our DB.


## 4. Deploy your webapp

#### Step 1

I am going to deploy a [project](https://github.com/taimourz/Dawn-news-rewinded-12-years) I recently worked on. This will work for any project you have

`Go to AWS Console` > `Search “EC2” on the search bar` > `Go to EC2 and Click on Launch Instance`
 

![[aws1.png]]

#### Step 2

- Select a name any name 
- Select Ubuntu free-tier as OS Image. 
- Select Instance Type which is under free-tier i.e t2-micro

![[aws2.png]]


#### Step 3

- Scroll down and click on Create new key pair.
- If you have any existing one then you can assign that but we don’t have so we will generate a new key pair and download it.
> SSH will happen only in the same folder on your laptop where your key pair is present.

![[aws3a.png]]

#### Step 4

- Give a name to key-pair and click on “Create Key pair” button.

![[aws3b.png]]

#### Step 5

- Scroll down and select “Create security group”.
- It will create a new security group.
- If you have any existing Security Group created then you can assign that. But we don’t have so we will create one.
-  Give “Allow SSH traffic from anywhere” access. It will permit to do SSH from any laptop anywhere.

#### Step 6

- Scroll down and configure storage settings.
> By default, it is given 8 GB storage so we will go with it.

![[aws3c.png]]

#### Step 7

- Wait for 2–3 minutes then your instance state will come under the “Running” State. 
- we have created your first EC2 instance. 

#### Step 8a: Directly connect

There are 2 ways to connect with you EC2 machine. 
- you can directly access your instance from the AWS console then you can click on “Connect” button under “EC2 Instance Connect”

![[aws4a.png]]


![[aws5.png]]

#### Step 8b: SSH

Or we do SSH from our laptop into this instance.
- Make sure that .pem file is available in current directory
- Then copy paste the given commands on terminal

![[aws6a.png]]
![[aws7.png]]

> Now we have successfully done SSH. Our terminal became the terminal of EC2. You can verify the IP address “172:31:15:226” of the below image to the private IP address of our instance that is visible in the image of


#### Step 9

Now that we have a brand-new machine. 
- We will do the project setup like we normally do our computers. For me i have to install node, npm
- Finally i start the application and it starts running on port 5173
``` bash
sudo apt update
sudo apt install nodejs
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 22
git clone https://github.com/taimourz/Dawn-news-rewinded-12-years.git
cd Dawn-news-rewinded-12-years/
npm i
echo "VITE_TAIMOUR_API_KEY=xxxxxxx" > .env
npm run dev
```


![[aws7b.png]]


#### Step 10

Now we need to expose Port 5173 to “Allow Anywhere”. 

> Initially, We only did SSH allow anywhere.

 - Select your running instance. 
 - Go to “Security” tab in the footer. Then select the security group link.


![[aws9.png]]

#### Step 11

You can see, currently, only ssh is allowed. Click on the “Edit inbound rules” button.

![[aws10a.png]]

#### Step 12

- Add a new rule that “Custom TCP” is allowed from anywhere on the internet on Port 5173

![[aws11.png]]

#### Step 13

- Again visit “http://<your_instance_public_IP>:8080” and boom, your website is live on the internet. And you can share it with anyone.

![[aws12.png]]

## 5. Destroy the EC2 Instance

- Terminate the instance that you have created. Otherwise, you might get a bill if you are not in the free-tier.

![[aws13.png]]

