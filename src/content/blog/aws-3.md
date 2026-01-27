---
title: Understanding Load Balancing and Scaling in AWS
date: 2026-1-28
tags: aws
---

## Contents

1. [Introduction](#1-introduction)
   - [Vertical Scaling](#vertical-scaling)
   - [Horizontal Scaling](#horizontal-scaling)
2. [Load Balancer & Target Groups](#2-load-balancer-and-target-groups)
   - [Target Group](#target-group)
   - [Features of ALB](#features-of-alb)
3. [Tutorial](#3-tutorial)
   - [Setup EC2 Instances](#1-setup-ec2-instances)
   - [Create Target Groups](#2-create-target-groups)
   - [Create Load Balancer](#3-create-load-balancer)
   - [Configure Security Groups](#4-configure-security-groups)
   - [Test Load Balancer](#5-test-load-balancer)
   - [Configure Routing Rules](#6-configure-routing-rules)
4. [Monitoring Traffic](#5-monitoring-traffic)
5. [Cleanup all resources](#6-cleanup-all-resources)

---

## 1. Introduction


Scaling means we need to increase the specs of our EC2 instance (like increasing RAM, CPU, storage etc) or add more EC2 instances to handle the load.

There are 2 types of scaling

1. Vertical Scaling
2. Horizontal Scaling

### Vertical Scaling

If we increase the specs (RAM, Storage, CPU) of the same machine to handle more load then it is called vertical scaling.

**Eg**: moving from "t2.micro" instance type to "t2.medium" machine

> **Use Case**: This type of scaling is mostly used in SQL databases and also in stateful applications because it is difficult to maintain consistency of states in a horizontal scaling setup.

### Horizontal Scaling

If we add more machines and distribute the incoming load then it is called horizontal scaling.

In this setup, Clients don't make requests directly to the server, instead, they send requests to the load balancer. The load balancer takes the incoming traffic and transfers it to the least busy machine.

Most of the time in the real world, we use horizontal scaling.

> **Use Case**: Ideal for stateless applications and microservices architecture where you can easily distribute traffic across multiple instances.

---

## 2. Load Balancer and Target Groups

We have already studied the role of load balancers. It takes the incoming request and sends it to the least busy server.

AWS provides 3 types of load balancers:

1. Application Load Balancer (ALB)
2. Network Load Balancer (NLB)
3. Classic Load Balancers (CLB)

We will only study ALB because it is the most used load balancer in the industries. NLB and CLB are rarely used.

### Target Group

ALB doesn't send the request directly to the EC2 instance. It sends the request to the target group. If we want this request to reach the EC2 instance, then we need to attach this target group to the EC2 instance.

Target Group TG1 is attached to the application load balancer, so it is sending all the incoming requests to the EC2 instances which has Target Group TG1 attached.

### Features of ALB

- In one ALB, you can attach more than one target group.
- ALB can route requests based on different criteria like host headers, path, source IP, etc.

> **Example**: You can set a rule like this:
> - `/api/*` forward to the Backend API target group
> - `/admin/*` forward to the Admin Panel target group

These are helpful when you have microservices and different microservices are deployed in different EC2 instances. Then you attach different target groups to it and configure it in ALB based on subdomains, params etc of the URL.

When you configure ALB then it gives you a domain name (not an IP address). Clients hit this ALB domain name on their browsers to access your website. As I told before, client don't directly hit `http://<website_ip_address>:3000`. Clients hit ALB and ALB will route their request to the least busy EC2 Instance.

---

## 3. Tutorial

1. launch 5 EC2 instances and deploy our Task-Management-CRUD application on each instance.
2. Create two target groups: TG-webapp and TG-node, and attach instances to them.
3. Create an Application Load Balancer and attach both target groups to it.
4. Configure security groups so that traffic only flows through the load balancer.
5. Configure routing rules so that requests are routed to the appropriate target group based on the path.

> Doing this will help us understand how to distribute incoming traffic across multiple EC2 instances using a load balancer, ensuring high availability and better performance for our application.

---

### 1. Setup EC2 Instances

Launch 5 EC2 instances. I have shown how to create EC2 instances [here](aws-1#4-deploy-your-webapp). Make sure all instances are in the same region.
SSh into each of the machine and do project setup. I am using this [Task-Management-CRUD application](https://github.com/taimourz/Task-Management-CRUD).

```bash

sudo apt update
sudo apt install nodejs
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 22
npm install -g pm2
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
   sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod

git clone https://github.com/taimourz/Task-Management-CRUD.git
cd Task-Management-CRUD
npm i

export PORT=3000
export MONGODB_URL=mongodb://127.0.0.1:27017/task-app-api
export JWT_SECRET=randomsecret
export SENDGRID_API_KEY=SG.79hafdFKss38cdSESYgqsBJEhbuA.F35EE4oK821nihl1MRknlGEY8q73aE1Lgosss2-n5vE4GI
# process manager to keep app running
pm2 start src/index.js --name task-manager-1
```

![[aws32.png]]


![[aws33.png]]

---

### 2. Create Target Groups

#### Step 1

- Go on Target groups from left sidebar
- Select **Instance** as the target type.
- Give a name **TG-webapp**.
- Set the port same as your application i.e 3000.
- You can use any endpoint for health check. I am using default i.e  `/` 

![[aws34.png]]

#### Step 2

- Select the EC2 instances which you want to attach to this TG-webapp Target Group.
- I have selected 3 instances for the webapp by clicking on "Include as pending below" button. 

![[aws35.png]]

#### Step 3

- Repeat the same process for the other Target group i.e TG-node

![[aws37.png]]

---

### 3. Create Load Balancer

#### Step 1

- Select `Load Balancers` from left sidebar.
- Give it a name, mine is `taimour-alb-tutorial`

![[aws38.png]]

![[aws39.png]]

#### Step 2

- In Network mapping section, select all three availability zones. This ensures high availability across multiple data centers.

![[aws40.png]]

#### Step 3

- Attach a default Target Group. The request will go to this target group when no rule such as `/api`, `/admin` etc matches.
- I have selected TG-webapp as the default target group.

![[aws41.png]]

---

### 4. Configure Security Groups

#### Step 1

- Copy the security group ID of the ALB.

![[aws44.png]]

#### Step 2

- [Edit inbound rules](aws-1#step-11) for each instance and add custom TCP rule on Port 3000.
- In source, paste the security group ID of the ALB 
> In this way only ALB can access our EC2 instance. Users cannot directly hit `http://<ip_address_of_instance>:3000` to get the response.

![[aws45.png]]

#### Step 3

- Edit inbound rules for ALB. 
- Use Port 80 and allow anywhere rule. This allows users to access the load balancer from the internet.

![[aws47.png]]

---

### 5. Test Load Balancer



Finally, You can see it gave us a DNS name (not an IP Address). You can hit this DNS name on your browser and the request will be redirected to the target group.

![[aws42.png]]


![[aws48.png]]

> You will not be able to hit `http://<ip_address_of_instance>:3000` because it doesn't have "allow anywhere" security rule. It only permits ALB traffic. So, you are hitting ALB and ALB routing your request to the respective instance which is very cool.

---

### 6. Configure Routing Rules

Now, we will set a rule so that requests with a specific path (like `/node/*`) will go to TG-node target group, while all other requests go to TG-webapp (the default target group).

#### Step 1

- Click on the HTTP:80 Link in "Listeners and rules" section to add a new rule.

![[aws49.png]]

#### Step 2

- Give any name.
- In conditions section, select path and add custom path
- At the end give it a priority to this rule. I have given 1. This way if paths are conflicting then higher priority will be choosen

> Since I have added `/node`, Now all the request with `/node` will go to TG-node target group.

> In the real world, what we do is, configure rules such that requests with `/` go in Home microservice, requests with `/api/*` go on API microservice, requests with `/admin/*` go on Admin Panel microservice and so on. I have shown the same thing in this tutorial. I hope you understand it.

![[aws50.png]]

![[aws51.png]]

#### Step 7

## 5. Monitoring Traffic

- Finally, Now when I go on path `/node`, It always redirects me to either `1-node` or `2-node` instance
- Similarly, when I go on path `/`, It always redirects me to `1-webapp`, `2-webapp` or `3-webapp` instance
- We can verify this by looking at request count chart which shows requests getting equally distributed between TG-webapp and TG-node.

![[aws52.png]]

## 6. Cleanup all resources

- Delete the ALB
- Delete Target Groups
- Terminate the EC2 instances
