# Documenting Project 7: Devops Tooling Website Solution

Step 1: Preparing NFS server
1. launching an ec2 instance with rhel os
2. creating 3 volumes and attaching them to the instance
3. downloading lvm and configuring the volumes into lv-apps, lv-logs, lv-opt
4. formatting disks as xfs
   `sudo mkfs -t xfs /dev/webdata-vg/lv-apps`
   ![formatting lv-apps](./Images-7/formatting%20lv-apps.PNG)
   `sudo mkfs -t xfs /dev/webdata-vg/lv-logs`
   ![formatting lv-logs](./Images-7/formatting%20lv-logs.PNG)
   `sudo mkfs -t xfs /dev/webdata-vg/lv-opt`
   ![formatting lv-opt](./Images-7/formatting%20lv-opt.PNG)
5. creating mount points
   `sudo mkdir /mnt/apps`
   `sudo mkdir /mnt/logs`
   `sudo mkdir /mnt/opt`
6. mounting logical volumes on the mount points
   `sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
   `sudo mount /dev/webdata-vg/lv-logs /mnt/logs`
   `sudo mount /dev/webdata-vg/lv-opt /mnt/opt`
7. installing NFS server
   `sudo yum -y update`
   `sudo yum install nfs-utils -y`
   `sudo systemctl start nfs-server.service`
   `sudo systemctl enable nfs-server.service`
   `sudo systemctl status nfs-server.service`
8. setting up permisions that will allow web server 777 on NFS server
   `sudo chown -R nobody: /mnt/apps`
   `sudo chown -R nobody: /mnt/logs`
   `sudo chown -R nobody: /mnt/opt`
   `sudo chmod -R 777 /mnt/apps`
   `sudo chmod -R 777 /mnt/logs`
   `sudo chmod -R 777 /mnt/opt`
   `sudo systemctl restart nfs-server.service`
9. Configuring access to NFS for clients within the same subnet
   `sudo vi /etc/exports`
   ![configuring access to nfs from client](./Images-7/configuring%20access%20to%20nfs%20from%20client.PNG)
   `sudo exportfs -arv`
   `rpcinfo -p | grep nfs`
   ![checking ports used by nfs](./Images-7/checking%20ports%20used%20by%20nfs.PNG)
10. opening ports TCP 111, TCP 2049, UDP 111, UDP 2049 in order for nfs to be accessible to client

Step 2: Configuring Database Server
1. Launching an ec2 instance with ubuntu
2. Installing MySQL server
   `sudo apt update`
   `sudo apt install mysql server`
   `sudo mysql`
   `create database tooling;`
   `create user 'webaccess'@'172.31.80.0/20' identified by 'password';`
   `grant all privileges on tooling.* to 'webaccess'@'172.31.80.0/20';`
   `flush privileges;`
   `show databases;`
   ![mysql databses](./Images-7/mysql%20database.PNG)
Step 3: Preparing the Web servers
1. Launching 3 ec2 instances with rhel os
2. Configuring nfs client
   `sudo yum update`
   `sudo yum install nfs-utils nfs4-acl-tools -y`
   `sudo systemctl start nfs.service`
   `sudo systemctl enable nfs.service`
   `sudo systemctl status nfs.service`
3. mounting /var/www and targeting nfs server's export for apps
   `sudo mkdir /var/www`
   `sudo mount -t nfs -o rw,nosuid 172.31.82.30:/mnt/apps /var/www`
4. updating fstab
   `sudo vi /etc/fstab`
   ![fstab update](./Images-7/fstab%20update.PNG)
5. Installing Remi repository, Apache and PHP
   `sudo yum install httpd -y`
   `sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm`
   `sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm`
   `sudo dnf module reset php`
   `sudo dnf module enable php:remi-8.2`
   `sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`
   `sudo systemctl start php-fpm`
   `sudo systemctl enable php-fpm`
   `setsebool -P httpd_execmem 1`
6. Verifying that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps, this confirms nfs is mounted correctly.
   `sudo ls /var/www`
   `sudo ls /mnt/apps`
7. mounting /var/log and targeting nfs server's export for logs
   `sudo mount -t nfs -o rw,nosuid 172.31.82.30:/mnt/logs /var/log/httpd`
   `sudo vi /etc/fstab`
8. forking the tooling source code from Darey.io
   `sudo yum install git`
   `sudo git init`
   `sudo git clone https://github.com/darey-io/tooling.git`
9. deploying html folder from repo to /var/www/html
   `sudo cp -R html/. /var/www/html`
10. opening port 80 on webserver
11. confirming response from apache on browser; disabling selinux
   `sudo setenforce 0`
12. making the above change persist
   `sudo vi /etc/sysconfig/selinux`
   ![selinux disabled](./Images-7/selinux%20disabled.PNG)
   [testing apache](https://54.144.159.250)
   ![apache test page successful](./Images-7/apache%20test%20page%20successful.PNG)
13. Updating the website’s configuration to connect to the database in functions.php
   `sudo vi /var/www/html/functions.php`
14. Applying tooling-db.sql script to my database
   `sudo yum install mysql`
   `sudo systemctl start mysqld`
   `sudo systemctl enable mysqld`
   `sudo systemctl status mysqld`
15. configuring binding address
   `sudo vi /etc/mysql/mysql.conf.d/mysqld.conf`
   `sudo systemctl restart mysqld`
   `mysql -h 172.31.80.14 -u webaccess -p <db-pasword> < tooling-db.sql`
16. Opening the website in my browser
   `sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup`
   [testing website](http://54.144.159.250/index.php)
   ![backup page created](./Images-7/backup%20page%20successfully%20created.PNG) 
17. logging into the website with 'admin'
   ![admin page](./Images-7/admin%20page1.PNG)
   ![admin page](./Images-7/admin%20page2.PNG)  


