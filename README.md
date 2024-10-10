# WEB-SOLUTION-WITH-WORDPRESS
Create an EC2 instance with the Redhat OS.
Name the instance Web-Server

Create three EBS volumes of 10 Gib each



![ebs volume creation](https://github.com/user-attachments/assets/1a5b8e2a-5a3c-4b19-b938-845f5ca0c6e0)

Next we connect our instance using the windows terminal




![connect to terminal](https://github.com/user-attachments/assets/ec5e729a-7c46-4b39-bc96-b56e66538345)


We can see the block device currently connected using the command:


    lsblk



    
![lsblk list block](https://github.com/user-attachments/assets/a7492248-9885-4888-b7c2-59899119cd16)

    
To see all the mounts and free spaces available on the server:
   
    
    
    
    df -h




    
![mounts and free spaces on the server](https://github.com/user-attachments/assets/b0903239-1bd5-4fbe-8aed-d1a1290315c1)

We use the gdisk utility to create single partition on each of the three disks. 
Starting with disk /dev/xvdf, enter the command:




    sudo gdisk /dev/xvdf



    
![to create single partition table on disk 1 contd](https://github.com/user-attachments/assets/d74e81a6-998f-499b-a4a9-d000e6c89700)

When asked for a command, type P for printing partition table, n for creating a new partion and W to write. Enter Yes to all and proceed with the process.




![to create single partition table on disk 1](https://github.com/user-attachments/assets/a5354f0a-9c1d-43f8-8b25-e89f8a915afe)

We repeat the process for disks /dev/xvdg and /def/xvdh




    sudo gdisk /dev/xvdg



![create partition xvdg](https://github.com/user-attachments/assets/09f120f2-744d-4c94-a3ae-f338722dac8d)

    

    sudo gdisk /dev/xvdh


![create partition xvdh](https://github.com/user-attachments/assets/d16f194b-d1c6-4fc5-96cb-3d5f32a87002)




5. Use lsblk to confirm new partition on each disk:

   
       lsblk

Now we can see the newly created partitions for all the disks



![newly created partitions](https://github.com/user-attachments/assets/1920265e-fdf9-46cd-9e9e-e756638e2f30)



6. Install the lvm2 package:


       sudo yum install lvm2



    ![installing lvm2](https://github.com/user-attachments/assets/d0bb99f8-96ef-4101-bfe1-8f0649be2823)



    ![installing lvm2 2](https://github.com/user-attachments/assets/c64aea52-b778-48c6-9dca-c80baf068f6d)



   Check for available partition:



       sudo lvmdiskscan

    ![sudo lvmdiskscan](https://github.com/user-attachments/assets/5a405105-1528-4537-a57e-fc1b987b151a)


 7. Use pvcreate utility to mark each of the three disks partitions to be used by 
    LVM:


        sudo pvcreate /dev/xvdf1

        sudo pvcreate /dev/xvdg1

        sudo pvcreate /dev/xvdh1




    ![sudo pvcreate to mark the 3 partitions as physical volumes used by lvm](https://github.com/user-attachments/assets/85e65fe0-d99a-4eb2-903a-5dc85a3b5d31)



 8. Verify the physical volume is created by entering:

    
         sudo pvs
![sudo pvs verify phusical volume created](https://github.com/user-attachments/assets/6a8eefc7-1302-4b09-aab0-ef7f1c18792f)

 10. Use Vg create utility to add all 3 physical volumes to a volume group. Lets call the new volume: webdata-vg:

     
         sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

 8. Verify the volume group is created by entering:
         sugo vgs

       
     ![sudo vgs](https://github.com/user-attachments/assets/191cbd50-7b0e-441a-b649-f8c069637299)

11. Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the
PV size), and logs-lv Use the remaining space of the PV size. NOTE:
apps-lv will be used to store data for the Website while, logs-lv will be
used to store data for logs.



      sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg  


![sudo lv create logs-lv](https://github.com/user-attachments/assets/d38bb034-4d82-41d5-9a0e-a1510b740e64)



12. Verify that your Logical Volume has been created successfully by
running


        sudo lvs

    ![sudo lvs](https://github.com/user-attachments/assets/c4c56df0-936f-4d94-b8db-ae7f9d4e692f)

           
13. Verify the entire setup

        sudo vgdisplay -v #view complete setup - VG, PV,         and LV

    ![sudo vgdisplay -v after creating vg pv lv ](https://github.com/user-attachments/assets/e93dd6b3-6f5c-49c3-b3b9-9af848395ee5)


        sudo lsblk

    
    ![sudo lsblk after creating vg pv and lv](https://github.com/user-attachments/assets/7e3c27f5-a1a6-4aa6-bc62-ca13b4d25a54)


15. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

    
        sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
        sudo mkfs -t ext4 /dev/webdata-vg/logs-lv


   ![sudo mkfs -t ext4 lv](https://github.com/user-attachments/assets/17f710ed-5a3c-4491-9dbb-aa8b94772826)

15. Create /var/www/html directory to store                 websitefiles



        sudo mkdir -p /var/www/html

    
    ![to create var www html dir for web files](https://github.com/user-attachments/assets/dcbc2093-aa8e-4d7f-8bc7-7347c79f3598)

18. Create /home/recovery/logs to store backup of log data



        sudo mkdir -p /home/recovery/logs

    
![create dir to store back up of log data](https://github.com/user-attachments/assets/aba6561f-bad6-416b-ae9a-f3db44f8a01a)


23. Mount /var/www/html on apps-lv logical volume

    
        sudo mount /dev/webdata-vg/apps-lv /var/www/html/


    ![mount varwwwhtml dir on app-lv logical volume](https://github.com/user-attachments/assets/4befbd7f-22e2-467d-96a1-eface3b6347a)


24. Use rsync utility to backup all the files in the 
   log directory /var/log into /home/recovery/logs 
  (This is required before mounting the file system)

     sudo rsync -av /var/log/ /home/recovery/logs/

 ![rsync utility to backup](https://github.com/user-attachments/assets/210456ba-8060-4d94-944c-993c34f5833f)

26. Mount /var/log on logs-lv logical volume. (Note that all the existing
data on /var/log will be deleted. That is why step 15 above is very
important)


    sudo mount /dev/webdata-vg/logs-lv /var/log
27. Restore log files back into /var/log directory

        sudo rsync -av /home/recovery/logs/ /var/log

    
    ![restore log files back into var-log directory](https://github.com/user-attachments/assets/baee434a-795e-4981-b40d-735ec9c7c963)


We can now see the mountpoints to show the position of the web app files and the log files using:


        lsblk


![lsblk to see mount points](https://github.com/user-attachments/assets/64e9c39e-6fe6-4431-9b17-edd58eb15b84)


      


28. Update /etc/fstab file so that the mount configuration will persist after
restart of the server.

The UUID of the device will be used to update the /etc/fstab file;


        sudo blkid

        
![sudo blkid - to get the uuid of device](https://github.com/user-attachments/assets/00b5e22a-bfa0-4db1-81e9-9b90fc676612)


        sudo nano /etc/fstab
        
Update /etc/fstab in this format using your own UUID and rememeber to
remove the leading and ending quotes.

        ![sudo nano -etc-fstab](https://github.com/user-attachments/assets/05b25edf-02d1-4d36-a735-52e15565d902)


    
32. Test the configuration and reload the daemon
    
        sudo mount -a
        sudo systemctl daemon-reload



34. Verify your setup by running df -h, output must look like this:


     ![df -h  after systemctl daemon reload](https://github.com/user-attachments/assets/4b7e6e09-13d7-4dac-b085-4bdcb83f8834)
#######################################################################################################################################################################################################################################################################################################################################################################################
**STEP 2**
# WEB-SOLUTION-WITH-WORDPRESS
Create an EC2 instance with the Redhat OS.
Name the instance Web-Server
![db server instance creation](https://github.com/user-attachments/assets/2ace169f-f21b-4228-a599-51e73f2f921f)

Create three EBS volumes of 10 Gib each


![db volume list](https://github.com/user-attachments/assets/dd9c9fcd-463f-4076-b5c5-0fca38ea5a59)


Next we connect our instance using the windows terminal






We can see the block device currently connected using the command:
    lsblk


![lsblk newly created block](https://github.com/user-attachments/assets/cc3cd401-b39f-411a-ade0-4ead0f82e39c)


    
To see all the mounts and free spaces available on the server:
    df -h
![df -h showing mounts and free spaces](https://github.com/user-attachments/assets/2f2ac5d8-e409-45ca-b751-d5d304445a63)

We use the gdisk utility to create single partition on each of the three disks. 
Starting with disk /dev/xvdf, enter the command:

    sudo gdisk /dev/xvdi


When asked for a command, type P for printing partition table, n for creating a new partion and W to write. Enter Yes to all and proceed with the process.




![sudo gdisk xvdi](https://github.com/user-attachments/assets/2598193d-4872-4139-b731-b4041e5a45db)


We repeat the process for disks /dev/xvdg and /def/xvdh



    sudo gdisk /dev/xvdj





 ![sudo gdisk dev-xvdj disk](https://github.com/user-attachments/assets/fe060333-0e79-4147-b8eb-59bda92e530b)


    sudo gdisk /dev/xvdh



![sudo gdisk xvdj](https://github.com/user-attachments/assets/4a6c6bb9-27de-47dc-a1c7-253147af60ed)



5. Use lsblk to confirm new partition on each disk:

   
       lsblk

Now we can see the newly created partitions for all the disks


![lsblk confirm created partitions](https://github.com/user-attachments/assets/7a2277b9-6263-4d74-b4e1-fffde180244f)

6. Install the lvm2 package:


       sudo yum install lvm2

   

   ![sudo yum install lvm2](https://github.com/user-attachments/assets/1e002a71-22e4-44ee-8652-d73d5046ab13)



   Check for available partition:



       sudo lvmdiskscan


![sudo lvm diskscan](https://github.com/user-attachments/assets/a2725345-1abd-4f4b-b31a-fd823b62e4b4)

 7. Use pvcreate utility to mark each of the three disks partitions to be used by 
    LVM:


        sudo pvcreate /dev/xvdf1

        sudo pvcreate /dev/xvdg1

        sudo pvcreate /dev/xvdh1




   ![pvcreate pv for the 3 disks](https://github.com/user-attachments/assets/cc91562d-a9c7-4cf2-964b-f1c9b57cc080)



 8. Verify the physical volume is created by entering

    
         sudo pvs

![sudo pvs](https://github.com/user-attachments/assets/a76227a9-90a2-460d-afea-4b9fe5ea6e4e)

 10. Use Vg create utility to add all 3 physical volumes to a volume group. Lets call the new volume: webdata-vg:

     
         sudo vgcreate webdata-vg /dev/xvdi1 /dev/xvdj1 /dev/xvdk1




 8. Verify the volume group is created by entering:
         sugo vgs

       
    ![sudo vgs for webdata-vg greated](https://github.com/user-attachments/assets/4e216fbd-1955-43c4-9c56-10fbb525a54a)



     
11. Use lvcreate utility to create 2 logical volumes. db-lv (Use half of the
PV size), and logs-lv Use the remaining space of the PV size. NOTE:
apps-lv will be used to store data for the Website while, logs-lv will be
used to store data for logs.



      sudo lvcreate -n db-lv -L 14G webdata-vg
      sudo lvcreate -n logs-lv -L 14G webdata-vg  




12. Verify that your Logical Volume has been created successfully by
running


        sudo lvs

   ![sudo lvs](https://github.com/user-attachments/assets/8acce0dd-4d89-4316-9627-24351268722d)


           
13. Verify the entire setup

        sudo vgdisplay -v #view complete setup - VG, PV,         and LV

    
![sudo vgdisplay -v n](https://github.com/user-attachments/assets/31743421-8692-4d05-9492-379bbe6f58e2)


        sudo lsblk

    
   ![lsblk to view our config so far](https://github.com/user-attachments/assets/3f12f12a-0502-4c0e-86c6-7b74cdf5aea2)



15. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

    
        sudo mkfs -t ext4 /dev/webdata-vg/db-lv
        sudo mkfs -t ext4 /dev/webdata-vg/logs-lv



15. Create /db directory to store websitefiles



        sudo mkdir -p /db




18. Create /home/recovery/logs to store backup of log data



        sudo mkdir -p /home/recovery/logs

    
![sudo home-recovery-logs for storing log data](https://github.com/user-attachments/assets/b6a5c985-517e-419f-80ec-8bdb51e04392)



23. Mount /db on db-lv logical volume

    
        sudo mount /dev/webdata-vg/db-lv /db

    We can view our configuration so far:


        lsblk



    
![lsblk to view our config so far png l](https://github.com/user-attachments/assets/d672b3d1-3264-4cda-88e2-3bf2e8b5f397)



25. Use rsync utility to backup all the files in the log directory /var/log into 
    /home/recovery/logs (This is required before mounting the file system)

     sudo rsync -av /var/log/ /home/recovery/logs/


See the content of the /var/log/ directory
    ![ls var log](https://github.com/user-attachments/assets/7b2f1e6e-6fe3-4b7f-8880-72d0fdb324ee)



    
![after rsnc utility to copy varlog to homeRecoveryLogs](https://github.com/user-attachments/assets/fdc2ae99-cf3b-4c1d-8c96-29e6d5c2c57c)


We can see that both directories are now identical in contents.



26. Mount /var/log on logs-lv logical volume. (Note that all the existing
data on /var/log will be deleted. This is why we copied to the /home/recovery/logs/ directory using the rsync utility


        sudo mount /dev/webdata-vg/logs-lv /var/log


    ![after mounting varlog dir to the log-lv](https://github.com/user-attachments/assets/a94b8493-aaa5-4a90-9f5a-4b2eac384286)

    
28. Restore log files back into /var/log directory. Now we have to copy back from the /home/recovery/logs/ directory to /var/log directory

        sudo rsync -av /home/recovery/logs/ /var/log

    
     ![varlog foler after recopying from homeRecovery](https://github.com/user-attachments/assets/1f1240bc-9642-43d6-ae30-dadbafdc2fed)


We can now see the mountpoints to show the position of the web app files and the log files using:


        lsblk


![lsblk to view our config so far png l](https://github.com/user-attachments/assets/3e4b83db-abb4-48a3-8b87-7dec0e1b90bc)




      


28. Update /etc/fstab file so that the mount configuration will persist after
restart of the server.

The UUID of the device will be used to update the /etc/fstab file;


        sudo blkid

![sudo blkid n](https://github.com/user-attachments/assets/160b4afd-3609-4960-bbd9-b4effd41112d)





        sudo nano /etc/fstab
        
Update /etc/fstab in this format using your own UUID and rememeber to
remove the leading and ending quotes.

![sudo etc fstab](https://github.com/user-attachments/assets/94411524-c879-4214-8522-7e446690c662)


    
32. Test the configuration and reload the daemon
    
        sudo mount -a
        sudo systemctl daemon-reload



34. Verify your setup by running df -h, output must look like this:

    ![df -h after daemon reload png n](https://github.com/user-attachments/assets/8f31d0c6-93f0-4b16-9a77-44eb1765a525)

    
############################################################

**STEP 3: Install WordPress on your Web Server EC2**    
1. Update the repository



        sudo yum -y update

   ![sudo yum -y update](https://github.com/user-attachments/assets/e56e96cd-bcff-44f9-bcfa-bd95d3b2b155)

3. Install wget, Apache and its dependencies

   
        sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

   ![installing wget apache and others](https://github.com/user-attachments/assets/11c790b1-26f3-4778-bc9a-4cefc69443a0)

5. Start Apache

   
        sudo systemctl enable httpd
        sudo systemctl start httpd

   ![enable and starting apache](https://github.com/user-attachments/assets/6e0d8de9-c768-4d69-8e0b-4d396de48cfd)


7. To install PHP and its dependencies

   
        sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
        sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
        sudo yum module list php
        sudo yum module reset php
        sudo yum module enable php:remi-7.4
        sudo yum install php php-opcache php-gd php-curl php-mysqlnd
        sudo systemctl start php-fpm
        sudo systemctl enable php-fpm
   
        #modify the security context of the Apache HTTP server process on a Linux system.
        sudo setsebool -P httpd_execmem 1

**Explanation of command**

**setsebool**: This is the command used to modify Security-Enhanced Linux (SELinux) boolean values. SELinux is a security module for Linux that provides a mandatory access control (MAC) system.

**-P**: This option indicates that the change should be made persistent, meaning it will be saved after the system is restarted.

**httpd_execmem:** This is the specific boolean value being modified. It controls whether the Apache HTTP server process is allowed to execute code from memory.

**1:** This sets the boolean value to 1, which means the Apache HTTP server process is allowed to execute code from memory.  



![installing fedora](https://github.com/user-attachments/assets/bc8737e7-2bb7-47ee-aef4-349a48e544a1)


9. Restart Apache using the command:

    
        sudo systemctl restart httpd
   
11. Next we will download wordpress and copy wordpress to /var/www/html

        #Create and enter the wordpress directory
        mkdir wordpress
        cd wordpress
    
        #Download the latest wordpress from the internedownloads the latest version of WordPress as a compressed archive file (tar.gz) from the official WordPress website
        sudo wget http://wordpress.org/latest.tar.gz
    
    ![content of wordpress directory](https://github.com/user-attachments/assets/724b98a7-dcba-444d-a110-57fe93388f5d)

        #Extract the downloaded archive file ("latest.tar.gz")
        sudo tar -xzvf latest.tar.gz

    ![wordpress](https://github.com/user-attachments/assets/24fa66c3-b4c2-4254-8f54-c2cb1a6d330f)

        #Delete the downloaded archive file "latest.tar.gz" after extraction
        sudo rm -rf latest.tar.gz

        #Copy the file "wp-config-sample.php" located inside the extracted "wordpress" directory to a new file named "wp-config.php" within the same directory:
    
        sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php

    ![contents of wordpress](https://github.com/user-attachments/assets/0d5ba3d4-e9a9-4604-ba6a-cbde20ca32de)

    ![wordpress content after copy config -sample to config](https://github.com/user-attachments/assets/e2efa5c0-7f6d-4145-b110-77bdd501c4f1)

        #Copy the entire extracted "wordpress" directory recursively (including all subdirectories and files) to the "/var/www/html/" directory.


        cp -R wordpress /var/www/html/
    
    Now check the content of the /var/www/html/
    
![ls var www html after copying](https://github.com/user-attachments/assets/92a1d0c2-00f4-4751-943d-4906c03fd31a)

    
13. Configure SELinux Policies


        #Change the ownership of the /var/www/html/wordpress directory and all its contents to the apache user and group.
        sudo chown -R apache:apache /var/www/html/wordpress

        #Change the ownership of the /var/www/html/wordpress directory and all its contents to the apache user and group. -t httpd_sys_rw_content_t sets the context to httpd_sys_rw_content_t, which grants read and write permissions to the httpd service.
        sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R


        #Enable network connectivity for the httpd service
        sudo setsebool -P httpd_can_network_connect=1

    ![configuring SElinux policies output](https://github.com/user-attachments/assets/583d5143-e345-4762-a9ba-4b2f8cef7a49)


Step 4 — Install MySQL on your DB Server EC2
sudo yum update
sudo yum install mysql-server
Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running,
restart the service and enable it so it will be running even after reboot:
sudo systemctl restart mysqld
sudo systemctl enable mysqld
Step 5 — Confi gure DB to work with WordPress
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
Step 6 — Confi gure WordPress to connect to remote database.
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow
access to the DB server ONLY from your Web Server's IP address, so in the Inbound Rule
configuration specify source as /32
34% COMPLETE 32/94 Steps  Previous Lesson

1. Install MySQL client and test that you can connect from your Web Server to your DB server by
using mysql-client
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
2. Verify if you can successfully execute SHOW DATABASES; command and see a list of existing
databases.
3. Change permissions and configuration so Apache could use WordPress:
4. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from
everywhere 0.0.0.0/0 or from your workstation's IP)
5. Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP￾Address>/wordpress/
34% COMPLETE 32/94 Steps  Previous Lesson

Fill out your DB credentials:
34% COMPLETE 32/94 Steps  Previous Lesson

If you see this message - it means your WordPress has successfully connected to your remote
MySQL database
34% COMPLETE 32/94 Steps  Previous Lesson

Important: Do not forget to STOP your EC2 instances after completion of the project to avoid extra
costs.
CONGRATULATIONS!
You have learned how to configure Linux storage susbystem and have also deployed a full-scale
Web Solution using WordPress CMS and MySQL RDBMS!
