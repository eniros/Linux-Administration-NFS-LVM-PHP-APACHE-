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


