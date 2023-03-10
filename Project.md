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

We'll be using RedHat OS for this project In previous projects we used `Ubuntu`, but it is better to be well-versed with various Linux distributions, thus, for this projects we will use very popular distribution called `RedHat` (it also has a fully compatible derivative ??? CentOS) Note: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like `ec2-user@<Public-IP>`.

## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS ???WEB SERVER???.
### Step 1 ??? Prepare a Web Server
1. We will launch an EC2 instance that will serve as "Web Server". We will create 3 volumes in the same AZ as our Web Server EC2, each of 10 GiB and attach them to the instance.
We can learn how to add EBS volume to an EC2 instance [here](https://www.youtube.com/watch?v=HPXnXkBzIHw).

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-22%20at%2023.24.47.png)

Our instance was created in "us-east-1e" availability zone as we can see in the image below, afterward, we will click on `volumes` highlighted in orange under the elastic block store on the left.

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-22%20at%2023.49.55.png)

Remember, we will be creating 3 volumes in the same AZ as our Web Server EC2 i.e us-east-1e each of 10 GiB.
To do this, click on "Create volume"

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-22%20at%2023.51.30.png)

Afterward, we will select the size as 10Gib and the availability zone to be the same as that of our web server.

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2000.18.27.png)

Now, we will select our newly created volume in the list of created volumes and click on the "Actions" menu and afterward "Attach Volume"

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2000.25.38.png)

After clicking on the above, it leads us to the page in the image below where we select the instance we want to attach the volume to and click on the "attach volume" icon button.

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2000.28.46.png)

We will follow this process to create and attach two more volumes.

2. We will ssh into the instance to begin configuration

3. We will use `lsblk` command to inspect what block devices are attached to the server. Notice names of our newly created devices. All devices in Linux reside in `/dev/` directory. We will inspect it with `ls /dev/` and make sure we see all 3 newly created block devices there ??? their names will likely be xvdf, xvdg, xvdh.

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2013.03.24.png)

4. We will use gdisk utility to create a single partition on each of the 3 disks. For example, for xvdf we will run the following: 
`sudo gdisk /dev/xvdf`

when promted for an input enter `n`, then press enter button 4 times for defualt settings. Afterward, enter `p` and press enter. Enter `w` and press enter and finally, enter `y` and press enter. The image below shows how we did it. We will repeat this step for xvdg and xvdh.

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2013.15.37.png)

5. Now, we will use `lsblk` utility to view the newly configured partition on each of the 3 disks.The image below is the result.

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2014.11.23.png)

6. Install lvm2 package using `sudo yum install lvm2`. We will then run `sudo lvmdiskscan` command to check for available partitions

7. We will use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM by running the following:
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
8. We will verify that our Physical volume has been created successfully by running `sudo pvs`

9. Use vgcreate utility to add all 3 PVs to a volume group (VG). We will mame the volume group "webdata-vg"

` sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1 `

10. Verify that our VG has been created successfully by running `sudo vgs`

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2014.25.03.png)

11. We will use lvcreate utility to create 2 logical volumes. `apps-lv` (Use half of the PV size), and `logs-lv` (Use the remaining space of the PV(Physical Volume) size). NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
``` 
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg

```
12. We will verify that our Logical Volume has been created successfully by running `sudo lvs`

13. Verify the entire setup by running:

```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
The result of the latter command should look like this

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2014.47.15.png)

14. Use mkfs.ext4 to format the logical volumes with [ext4](https://en.wikipedia.org/wiki/Ext4) filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
15. We will create `/var/www/html` directory to store website files
`sudo mkdir -p /var/www/html`

16. Then, we will create /home/recovery/logs to store backup of log data
`sudo mkdir -p /home/recovery/logs`

17. Mount /var/www/html on apps-lv logical volume
`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

18. Use rsync utility to back up all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
`sudo rsync -av /var/log/. /home/recovery/logs/`

19. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)
`sudo mount /dev/webdata-vg/logs-lv /var/log`

20. Restore log files back into /var/log directory
`sudo rsync -av /home/recovery/logs/. /var/log`

21. We will update `/etc/fstab` file so that the mount configuration will persist after the restart of the server.
The UUID of the device will be used to update the /etc/fstab file;
`sudo blkid`

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2015.19.11.png)

From the image above, the UUID value of vg-apps and vg-logs can be seen.
`sudo vi /etc/fstab`

``` 
# mount for wordpress webserver
UUID=078697ff-962b-4411-90ef-56eb986a7956 /var/www/html  ext4 defaults 0 0
UUID=08274569-5e48-4549-af7d-040b1dafbec2 /var/log       ext4 defaults 0 0

```
![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2016.16.07.png)

To edit, press i and to exit press esc and type :wq

22. Test the configuration and reload the daemon
```
sudo mount -a 
sudo systemctl daemon-reload

```

23. We will verify our setup by running `df -h` the output must look like this: 

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-23%20at%2016.27.02.png)

### Step 2 ??? Prepare the Database Server
We will launch a second RedHat EC2 instance that will serve as ??? ???DB Server??? Repeat the same steps as for the Web Server, but instead of `apps-lv`, we will create `db-lv` and mount it to `/db` directory instead of `/var/www/html/`.

1. We will launch an EC2 instance that will serve as "DB Server". We will create 3 volumes in the same AZ as our Web Server EC2, each of 10 GiB and attach them to the instance.

2. We will ssh into the instance to begin configuration

3. We will use `lsblk` command to inspect what block devices are attached to the server. Notice names of our newly created devices. All devices in Linux reside in `/dev/` directory. We will inspect it with `ls /dev/` and make sure we see all 3 newly created block devices there ??? their names will likely be xvdf, xvdg, xvdh.

4. `df -h` 

5. We will use gdisk utility to create a single partition on each of the 3 disks. For example, for xvdf we will run the following: 
`sudo gdisk /dev/xvdf`

when promted for an input enter `n`, then press enter button 4 times for defualt settings. Afterward, enter `p` and press enter. Enter `w` and press enter and finally, enter `y` and press enter. The image below shows how we did it. We will repeat this step for xvdg and xvdh.

6. Now, we will use `lsblk` utility to view the newly configured partition on each of the 3 disks.

7. Install lvm2 package using `sudo yum install lvm2`. We will then run `sudo lvmdiskscan` command to check for available partitions
8. We will use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM by running the following:
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
9. We will verify that our Physical volume has been created successfully by running `sudo pvs`

10. Use vgcreate utility to add all 3 PVs to a volume group (VG). We will mame the volume group "db-vg"

`sudo vgcreate db-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

11. Verify that our VG has been created successfully by running `sudo vgs`

12. We will use lvcreate utility to create 1 logical volume. `db-lv` (we will use 25g)

``` 
sudo lvcreate -n db-lv -L 25G db-vg

```
13. We will verify that our Logical Volume has been created successfully by running `sudo lvs`

14. Verify the entire setup by running:

```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
15. We will use mkfs.ext4 to format the logical volumes with ext4 filesystem

``` 
sudo mkfs -t ext4 /dev/db-vg/db-lv
```
16. Create a /db directory and mount it:

```
sudo mkdir /db
sudo mount /dev/db-vg/db-lv /db
df -h

```
17. We will update `/etc/fstab` file so that the mount configuration will persist after the restart of the server.
The UUID of the device will be used to update the /etc/fstab file;
`sudo blkid`

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-24%20at%2010.07.10.png)

In the image above, the UUID value is highlighted.
`sudo vi /etc/fstab`

``` 
# mount database
UUID=1d162135-1feb-4e26-85be-38b47c04e196 /db  ext4 defaults 0 0

```
To edit, press i and to exit press esc and type :wq

18. Test the configuration and reload the daemon
```
sudo mount -a 
sudo systemctl daemon-reload
```
19. We will verify our setup by running `df -h`

### Step 3 ??? Install WordPress on your Web Server EC2
1. Update the repository

`sudo yum -y update`

2. Install wget, Apache and it???s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

3. Start Apache

```
sudo systemctl enable httpd 
sudo systemctl start httpd
```

4. To install PHP and its dependencies

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
```

5. Restart Apache

`sudo systemctl restart httpd`

6. Download wordpress and copy wordpress to var/www/html
```
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
```

6. Configure SELinux Policies
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db 1
```

### Step 4 ??? Install MySQL on your DB Server EC2
```
sudo yum update
sudo yum install mysql-server
```
We will verify that the service is up and running by using 
`sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
then, check the status again `sudo systemctl status mysqld`

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-24%20at%2012.22.25.png)

### Step 5 ??? Configure DB to work with WordPress

In the database server terminal, we'll do mysql secure installation
`sudo mysql_secure_installation`

Now, we will run `sudo mysql` remember to add -p since you put a new password in the previous command.

```
CREATE DATABASE wordpress;
CREATE USER 'myuser'@'<Web-Server-Private-IP-Address>' IDENTIFIED BY 'put your password';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit

```

Now, we will edit the bind address to have the webserver's private ip address.
`sudo vi /etc/my.cnf`

```
[webserver]
bind-address=<your webserver private ip address>

```
like this image:

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-24%20at%2012.26.15.png)

Restart mysql service

`sudo systemctl restart mysqld`

### Step 6 - Setting up Security Group

We will open MySQL on port 3306 on DB Server EC2. For extra security, we will allow access to the DB server ONLY from our Web Server???s IP address, so in the Inbound Rule configuration specify source as /32.

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-24%20at%2023.39.36.png)

Enable TCP port 80 in Inbound Rules configuration for our Web Server EC2 (enable from everywhere 0.0.0.0/0 or from our workstation???s IP)

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-24%20at%2023.49.09.png)


### Step 7 ??? Install MySQL on the Web Server

```
sudo yum update 
sudo yum install mysql-server
```
We will verify that the service is up and running by using

`sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:

```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
then, check the status again `sudo systemctl status mysqld`

Let's test that we can connect from our Web Server to our DB server by using mysql-client

`sudo mysql -u <your db user name> -p -h <DB-Server-Private-IP-address>`

Verify if we can successfully execute `SHOW DATABASES;` command and see a list of existing databases.

Now, we will locate and edit the `wp-config.php`
```
cd /var/www/html/wordpress/ 
ls -l 
sudo vi wp-config.php

```
Edit the wp-config.php file to contain details about the remote db i.e database_name, database_user, database_password and database_host (private ip of database server in this case)

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-24%20at%2023.17.49.png)

To disable the default apache homepage

`sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup`

```
sudo systemctl restart httpd 
sudo systemctl status httpd

```

We will try to access our browser the link to our WordPress:

`http://<Web-Server-Public-IP-Address>/wordpress/`

Fill out the required credentials and click on install wordpress. Login and finally take a deep breath and enjoy the beautiful view of the wordpress page you see. 

![](https://github.com/Omolade11/Web-Solution-using-WordPress-and-MySQL-/blob/main/Images/Screenshot%202023-02-25%20at%2000.02.48.png)


