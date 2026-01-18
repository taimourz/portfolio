---
title: AWS EBS Volumes
date: 2026-1-19
tags: aws
---

## Contents

1. [Introduction](#1-introduction)
   - [Instance Store](#instance-store)
   - [EBS Volumes](#ebs-volumes)
   - [Types of EBS Volumes](#types-of-ebs-volumes)
2. [Overview](#2-overview)
3. [Creating and Attaching EBS Volume to an EC2 instance](#3-creating-and-attaching-ebs-volume-to-an-ec2-instance)
4. [Changing existing EBS volume without downtime](#4-changing-existing-ebs-volume-without-downtime)
5. [Snapshots and Backups](#5-snapshots-and-backups)
   - [How to create a Snapshots](#how-to-create-a-snapshots)
6. [Cleanup all resources](#6-cleanup-all-resources)

--- 

## 1. Introduction

We use SSDs or HDD for storage in laptops and computers. Similarly, we EBS volume is nothing but storage we use for our EC2 instance. 

### Instance Store

This is the storage type that is physically attached to the host machine (EC2) running. Data on an instance store is lost when the instance stops or terminates.

- It's created inside the EC2 machine which means it is the part of the machine but the EBS volume is kept away from the EC2 machine
- Since it is physically attached, it gives very high performance because data packets don't have to travel from somewhere else to your EC2 instance.

> **Use Case**: Ideal for temporary storage of information that changes frequently, such as caches, buffers, or scratch data.


### EBS Volumes

On the other hand, EBS is like an external hard drive that is plugged into oour EC2 instance. This means you can attach or detach it to your instance anytime. Even if your instance gets terminated it won't affect your EBS volume and you can attach this EBS volume to some other EC2 instance.

- EBS volume is not created inside the EC2 instance. It sits outside within the same availability zone.
- It is is more reliable as its data persist even after our instance gets terminated.

> **Use Case**: It is suitable for databases, file systems, or any application that requires persistent storage.

---




### Types of EBS Volumes

AWS provides different kinds of EBS volume with different pricing and we can choose depending on our usecase

1. **General purpose SSD (gp2 and gp3)**: Suitable for any general task.
2. **Provisioned IOPS SSD (io1 and io2)**: High-performance SSD volumes designed for critical applications 
3. **Throughput Optimized HDD (st1)**: frequently accessed, throughput-intensive workloads.
4. **Cold HDD (sc1)**:  less frequently accessed workloads.

[See Details here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html).

---

## 2. Overview

1. We will launch our EC2 Instance  with a default EBS Volume of 9 GB
2. We will attach another external 4 GB EBS Volume and attach to this EC2 Instance 1.
3. Store some code from GitHub to this external EBS Volume of 4GB.
4. Launch a second EC2 Instance with a default  volume of 10 GB and attach the same previous EBS volume to the new EC2 instance 
5. Finally, verify if our stored data persists or not.


> By doing this exercise, we will understand that we can create EBS Volumes independently and attach them to any EC2 instance. And even if our EC2 instance gets terminated, it won't affect our EBS Volumes.

---

## 3. Creating and Attaching EBS Volume to an EC2 instance

#### Step 1

- Launch an EC2 instance. I have shown this [here](aws-1.md). My instance name is 'taimour-ebs' and I have selected 9 GB gp3 storage
> we have to create EBS volume in the same availability zone.

![[aws14.png]]




#### Step 2

- Go to `Elastic Block Store > Volumes`. We can see that we have a 9 GB volume already present, it was created while we were launching our instance.
- Next, We will create a new volume of 4GB of type gp2 in the same availability zone i.e ap-south-1a
> Note: When we will terminate our instance then this 9 GB volume will get deleted because we haven't configured it to not delete while launching the instance. If you don't want your default EBS volume to be deleted with your instance, we have to go on "advanced" and uncheck the select box of delete.

![[aws15.png]]


![[aws16.png]]


#### Step 3

- Once created, we need to attach the volume to our EC2 instance (in the same availability zone). For me its 'taimour-ebs' we created above
- we can select any device type (doesn't matter)

![[aws17.png]]
![[aws18.png]]


#### Step 4


- Now, that "taimour-ebs-volume" is attached to the instance "taimour-ebs". we can ssh into our EC2 instance
- First we need to check, what block devices (hard disk, EBS Volumes) etc attached to this machine.
- we can see our default (xvda 9GB) and newly created volume (xvdd 4GB)
- Make a folder and mount that file system into this folder. Now all the things that you download or make in this folder will be created in the file system of EBS Volume.

> First, we have to enable the File System for newly created xvdf storage. If you are attaching any existing EBS volume then you don't need to do this. But, since we created this EBS Volume fresh and haven't enabled the file system till now, we have to do it.




```bash
# check block devices (hard disk, EBS Volumes) etc attached to this machine.
lsblk
# To check file system is enabled or not
sudo file -s /dev/xvdf
# If the response comes as "/dev/xvdf: data", then it is not enabled.
# Enable File System
sudo mkfs -t xfs /dev/xvdf
# Finally, check if volume is enabled or not
df -k
# Mount the file system of "xvdf" into this folder
mkdir myFileStorage
sudo mount /dev/xvdf myFileStorage
# store some data for testing. i am cloning one of my repos
git clone https://github.com/taimourz/excalidraw-url-exporter.git
sudo mv excalidraw-url-exporter/ myFileStorage
```

![[aws20.png]]


#### Step 5

- Terminate your instance
![[aws21.png]]


## 4. Changing existing EBS volume without downtime

#### Step 1
- Launch a new EC2 Instance. I named mine "taimour-ebs2" with default root EBS Volume 10 GB.
- Similarly, attach the same old volume to this new instance

![[aws22.png]]
![[aws23.png]]



#### Step 2

- Go to `Modify volume` then give the new volume size.
> **Note**: You can only increase the size, which means if you have initially given 4 GB then you can only give a size which is greater than 4GB.

![[aws25.png]]
![[aws26.png]]




#### Step 3
- SSH in the new EC2 instance "taimour-ebs2" and run lsblk to check if the volume is correctly attached or not.
- Since we already created the file system of this EBS Volume so we don't need to do it now. Just make any folder in this EC2 instance and attach this file system to that folder. Then checkA if the code that we installed previously persists or not.





```bash
# check block devices (hard disk, EBS Volumes) etc attached to this machine.
lsblk
# First, check what filesystem type you're using:
sudo file -s /dev/xvdf
# If it shows XFS filesystem, use:
sudo xfs_growfs /dev/xvdf
# If it shows ext4 filesystem, use:
sudo resize2fs /dev/xvdf

mkdir taimourEbs2InstanceFileStorage
sudo mount /dev/xvdd taimourEbs2InstanceFileStorage
df -k
cd taimourEbs2InstanceFileStorage
# we get the same data that we downloaded previously. This shows data persists.
ls
```
![[aws27.png]]
![[aws31.png]]



## 5. Snapshots and Backups

Snapshots are used to store a previous state of our storage. Just like we can use snapshots to create EBS volumes
And it will have all the data up to that point when we capture the snapshot. So, snapshots are also used to maintain the backup of the data.

There is another very cool advantage of snapshots. They are region-specific. This means a snapshot created from ap-south-1a, ap-south-1b, all will be created in the region ap-south-1. And you can create EBS Volume with the help of the snapshot in any availability zone of the snapshot region.

> **Example**: Suppose you want to attach an EBS Volume of ap-south-1a to an EC2 instance of ap-south-1b. What you can do is, create the snapshot of ap-south-1a EBS volume, then launch another EBS Volume in the zone ap-south-1b with the help of the snapshot then attach it to the instance which is running on ap-south-1b.

### How to create a Snapshots

#### Step 1

- We will create a snapshot from our same EBS Volume "taimour-ebs-volume". Since, that volume was present on ap-south-1a, so our snapshot will be created at the ap-south-1 region.
- Add description. Generally, people give the date as the description

![[aws28.png]]
![[aws29.png]]
![[aws30.png]]



## 6. Cleanup all resources

- Before deleting EBS Volume, make sure to detach it first if it's attached to any instance.
- Then delete EBS Volume.
- Delete Snapshot.
- Terminate the EC2 Instances that we created.