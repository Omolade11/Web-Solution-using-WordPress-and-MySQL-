# Full-scale Web Solution using WordPress CMS and MySQL RDBMS

In this project, we will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. 
WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

This project consists of two parts:
1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give  practical experience of working with disks, partitions and volumes in Linux.
2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify skills of deploying Web and DB tiers of Web solution.

## Three-tier Architecture
Generally, web, or mobile solutions are implemented based on what is called the "Three-tier Architecture".
Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.
![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/3tier%20app.webp)

1. Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
2. Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
3. Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server.

In this project, we will showcase Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.
We will be working with several storage and disk management concepts.

Our 3-Tier Setup
1. A Laptop or PC to serve as a client
2. An EC2 Linux Server as a web server (This is where we will install WordPress)
3. An EC2 Linux server as a database (DB) server

We'll be using RedHat OS for this project In previous projects we used `Ubuntu`, but it is better to be well-versed with various Linux distributions, thus, for this projects we will use very popular distribution called `RedHat` (it also has a fully compatible derivative – CentOS) Note: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like `ec2-user@<Public-IP>`.

## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.
### Step 1 — Prepare a Web Server
We will launch an EC2 instance that will serve as "Web Server". We will create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
We can learn how to add EBS volume to an EC2 instance [here](https://www.youtube.com/watch?v=HPXnXkBzIHw).

[](Images/Screenshot 2023-02-22 at 23.24.47.png)


