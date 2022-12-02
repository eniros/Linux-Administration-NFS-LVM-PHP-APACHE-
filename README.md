# Linux-Administration-NFS-LVM-PHP-APACHE-
Deploying DEVOPS TOOLING

STEP 1 – Create NFS SERVER
Launch an EC2 Instance and add 3 EBS Blocks
Create and partition the volume groups
Create 3 logical volumes

<img width="570" alt="Screenshot 2022-12-01 at 21 30 21" src="https://user-images.githubusercontent.com/61475969/205163242-fbc36475-7a26-4d79-b25e-2d387dedfc14.png">

Change file format to xfs with the command;
sudo lvcreate -n lv-apps -L 9G webdata-vg 
<img width="570" alt="Screenshot 2022-12-01 at 21 34 38" src="https://user-images.githubusercontent.com/61475969/205163983-2f1c02b5-39f2-4904-b45b-8d92869e32bf.png">

Create mount directories

sudo mkdir /mnt/apps

<img width="570" alt="Screenshot 2022-12-01 at 21 37 08" src="https://user-images.githubusercontent.com/61475969/205164277-50f1144c-e33e-4929-b705-c7cebb7d0517.png">

Mount Volumes into directories
sudo mount /dev/webdata-vg/lv-apps /mnt/apps

<img width="570" alt="Screenshot 2022-12-01 at 21 39 33" src="https://user-images.githubusercontent.com/61475969/205164656-6f0e2926-1d93-4180-bea8-6477bafe5ba4.png">

Install NFS server and configure it to start on reboot by running the following commands;

sudo apt-get update
sudo apt install nfs-kernel-server
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
<img width="570" alt="Screenshot 2022-12-01 at 21 46 03" src="https://user-images.githubusercontent.com/61475969/205165724-bfb949ed-1d69-428b-ad98-27c3515e2ba1.png">

Step 2 - Launch the DB Server

Launch another instance to use as server end and install mysql server
Create Database mysql
Create user in mysql
Grant privileges and update

<img width="570" alt="Screenshot 2022-12-01 at 22 08 16" src="https://user-images.githubusercontent.com/61475969/205169450-569c4018-1ada-4171-8df7-8b5c73235878.png">

Export the mounts for webservers subnet cidr to connect as clients. These webservers can be in the same subnet or in different subnets. For test purposes, we used same subnets but for production we would want to separate each tier inside its own subnet for higher level of security.


 Set up permission that will allow our Web servers to read, write and execute files on NFS by running the commands;
 sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service

<img width="570" alt="Screenshot 2022-12-01 at 22 15 08" src="https://user-images.githubusercontent.com/61475969/205170524-2f19e64a-a953-41fa-a9f5-7035e21921ac.png">

 Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ) by editing the config file
 
 sudo vi /etc/exports
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

<img width="570" alt="Screenshot 2022-12-01 at 22 25 18" src="https://user-images.githubusercontent.com/61475969/205172199-ffa4fa03-63a7-486e-a820-32ca397472ff.png">

 Export the changes made by running the command;
 sudo exportfs -arv
 
 Check which port is used by NFS by running the command (and open it using Security Groups -add new Inbound Rule -on the NFS server)
  
 rpcinfo -p | grep nfs
  
 <img width="570" alt="Screenshot 2022-12-01 at 22 33 03" src="https://user-images.githubusercontent.com/61475969/205173236-446d85cf-eec7-49e8-8013-7bf44f02a986.png">
  
 <img width="1154" alt="Screenshot 2022-12-01 at 22 40 27" src="https://user-images.githubusercontent.com/61475969/205174339-6427bc4c-187d-42dc-a541-0e5bb6e6da08.png">


 Step 3- Prepare the Web Server
 
 Launch an EC2 instance (ubuntu) to server as the nfs client
 Install the NFS Client by running the command:
  sudo apt install nfs-common

  <img width="563" alt="Screenshot 2022-12-01 at 22 59 00" src="https://user-images.githubusercontent.com/61475969/205176868-8d3ead41-c7a7-463c-b177-cf27bf2c77e0.png">

 Mount /var/www and target the NFS server's exports for apps:
 sudo mount -t nfs -o rw,nosuid 44.202.28.61:/mnt/apps /var/www
  
  
  Verify that the mount was successful and also update the etc/fstab file to make sure the mount stays intact after reboot.
  df -h sudo vi /etc/fstab :/mnt/apps /var/www nfs defaults 0 0
  
 ****add image
  
  Install Remi’s repository, Apache,PHP and its dependencies
  
Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps.
You can verify by creating a file on webserver 1 and check webserver 2 if its available.
  
sudo mount -t nfs -o rw,nosuid :/mnt/logs /var/log
  
Also update the etc/fstab file also on each webservers

Deploy the code to /var/www/html folder in one of the webservers.

You can do this by installing git on one of the servers and clone the repo into the folder
  
Update the website’s configuration to connect to the database (in /var/www/html/functions.php file). Apply tooling-db.sql script to your database using this command 

<img width="575" alt="Screenshot 2022-12-01 at 23 58 44" src="https://user-images.githubusercontent.com/61475969/205184373-f5a39f0a-ae72-4671-9c60-e9b869c69227.png">
  
Open a Mysql port 3306 for webservers in the subnet cidr where the webservers are located.</b>

Create in MySQL a new admin user with username and password
 
  Insert new user into the database (You can also do this by editing the tooling-db.sql script appropriately.) (INSERT INTO ‘users’ (’id’, ‘username’, ‘password’, ‘email’, ‘user_type’, ‘status’) VALUES -> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);)

<img width="954" alt="Screenshot 2022-12-02 at 00 10 55" src="https://user-images.githubusercontent.com/61475969/205185575-6facde40-cba7-4a33-81d7-c9566c646fcc.png">

  
