# WEB-SOLUTION-WITH-WORDPRESS


**STEP 1: Prepare the Web-Server**


1. Create an EC2 instance with the Redhat OS.
2. Name the instance Web-Server

3. Create three EBS volumes of 10 Gib each and attach these volumes to the ec2 instance.



![ebs volume creation](https://github.com/user-attachments/assets/1a5b8e2a-5a3c-4b19-b938-845f5ca0c6e0)


4. Next we connect our instance using the windows terminal




![connect to terminal](https://github.com/user-attachments/assets/ec5e729a-7c46-4b39-bc96-b56e66538345)


5. We can see the block device currently connected using the command:


        lsblk



    
![lsblk list block](https://github.com/user-attachments/assets/a7492248-9885-4888-b7c2-59899119cd16)

    
6. To see all the mounts and free spaces available on the server:
   
    
    
    
        df -h




    
![mounts and free spaces on the server](https://github.com/user-attachments/assets/b0903239-1bd5-4fbe-8aed-d1a1290315c1)

7. We use the gdisk utility to create single partition on each of the three disks. 
   Starting with disk /dev/xvdf, enter the command:




        sudo gdisk /dev/xvdf



    
![to create single partition table on disk 1 contd](https://github.com/user-attachments/assets/d74e81a6-998f-499b-a4a9-d000e6c89700)

8. When asked for a command, type P for printing partition table, n for creating a new partion and W to write. Enter Yes to all and proceed with the process.




![to create single partition table on disk 1](https://github.com/user-attachments/assets/a5354f0a-9c1d-43f8-8b25-e89f8a915afe)

9. We repeat the process for disks /dev/xvdg and /def/xvdh




        sudo gdisk /dev/xvdg



![create partition xvdg](https://github.com/user-attachments/assets/09f120f2-744d-4c94-a3ae-f338722dac8d)

    

        sudo gdisk /dev/xvdh


![create partition xvdh](https://github.com/user-attachments/assets/d16f194b-d1c6-4fc5-96cb-3d5f32a87002)




10. Use lsblk to confirm new partition on each disk:

   
       lsblk

11. Now we can see the newly created partitions for all the disks



![newly created partitions](https://github.com/user-attachments/assets/1920265e-fdf9-46cd-9e9e-e756638e2f30)



12. Install the lvm2 package:


       sudo yum install lvm2



    ![installing lvm2](https://github.com/user-attachments/assets/d0bb99f8-96ef-4101-bfe1-8f0649be2823)



    ![installing lvm2 2](https://github.com/user-attachments/assets/c64aea52-b778-48c6-9dca-c80baf068f6d)



13. Check for available partition:



           sudo lvmdiskscan

    ![sudo lvmdiskscan](https://github.com/user-attachments/assets/5a405105-1528-4537-a57e-fc1b987b151a)


 14. Use pvcreate utility to mark each of the three disks partitions to be used by 
    LVM:


            sudo pvcreate /dev/xvdf1
    
            sudo pvcreate /dev/xvdg1
    
            sudo pvcreate /dev/xvdh1




![sudo pvcreate to mark the 3 partitions as physical volumes used by lvm](https://github.com/user-attachments/assets/85e65fe0-d99a-4eb2-903a-5dc85a3b5d31)



 15. Verify the physical volume is created by entering:

    
         sudo pvs
![sudo pvs verify phusical volume created](https://github.com/user-attachments/assets/6a8eefc7-1302-4b09-aab0-ef7f1c18792f)

 16. Use Vg create utility to add all 3 physical volumes to a volume group. Lets call the new volume: webdata-vg:

     
         sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

 17. Verify the volume group is created by entering:
         sugo vgs

       
     ![sudo vgs](https://github.com/user-attachments/assets/191cbd50-7b0e-441a-b649-f8c069637299)

18. Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the
    PV size), and logs-lv Use the remaining space of the PV size. NOTE:
    apps-lv will be used to store data for the Website while, logs-lv will be
    used to store data for logs.



          sudo lvcreate -n apps-lv -L 14G webdata-vg
          sudo lvcreate -n logs-lv -L 14G webdata-vg  


![sudo lv create logs-lv](https://github.com/user-attachments/assets/d38bb034-4d82-41d5-9a0e-a1510b740e64)



19. Verify that your Logical Volume has been created successfully by
running


        sudo lvs

    ![sudo lvs](https://github.com/user-attachments/assets/c4c56df0-936f-4d94-b8db-ae7f9d4e692f)

           
20. Verify the entire setup

        sudo vgdisplay -v #view complete setup - VG, PV,         and LV

    ![sudo vgdisplay -v after creating vg pv lv ](https://github.com/user-attachments/assets/e93dd6b3-6f5c-49c3-b3b9-9af848395ee5)


        sudo lsblk

    
    ![sudo lsblk after creating vg pv and lv](https://github.com/user-attachments/assets/7e3c27f5-a1a6-4aa6-bc62-ca13b4d25a54)


21. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

    
        sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
        sudo mkfs -t ext4 /dev/webdata-vg/logs-lv


   ![sudo mkfs -t ext4 lv](https://github.com/user-attachments/assets/17f710ed-5a3c-4491-9dbb-aa8b94772826)

22. Create /var/www/html directory to store                 websitefiles



        sudo mkdir -p /var/www/html

    
    ![to create var www html dir for web files](https://github.com/user-attachments/assets/dcbc2093-aa8e-4d7f-8bc7-7347c79f3598)

23. Create /home/recovery/logs to store backup of log data



        sudo mkdir -p /home/recovery/logs

    
![create dir to store back up of log data](https://github.com/user-attachments/assets/aba6561f-bad6-416b-ae9a-f3db44f8a01a)


24. Mount /var/www/html on apps-lv logical volume

    
        sudo mount /dev/webdata-vg/apps-lv /var/www/html/


    ![mount varwwwhtml dir on app-lv logical volume](https://github.com/user-attachments/assets/4befbd7f-22e2-467d-96a1-eface3b6347a)


25. Use rsync utility to backup all the files in the 
   log directory /var/log into /home/recovery/logs 
  (This is required before mounting the file system)

     sudo rsync -av /var/log/ /home/recovery/logs/

 ![rsync utility to backup](https://github.com/user-attachments/assets/210456ba-8060-4d94-944c-993c34f5833f)

26. Mount /var/log on logs-lv logical volume. (Note that all the existing
data on /var/log will be deleted. That is why step 15 above is very
important)


     sudo mount /dev/webdata-vg/logs-lv /var/log
    
28. Restore log files back into /var/log directory

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
        
29. Update /etc/fstab in this format using your own UUID and rememeber to
    remove the leading and ending quotes.

![sudo nano -etc-fstab](https://github.com/user-attachments/assets/05b25edf-02d1-4d36-a735-52e15565d902)


    
30. Test the configuration and reload the daemon
    
        sudo mount -a
        sudo systemctl daemon-reload



31. Verify your setup by running df -h, output must look like this:


     ![df -h  after systemctl daemon reload](https://github.com/user-attachments/assets/4b7e6e09-13d7-4dac-b085-4bdcb83f8834)



**STEP 2: Prepare the Database Server**

1. Create an EC2 instance with the Redhat OS and name the instance Web-Server
![db server instance creation](https://github.com/user-attachments/assets/2ace169f-f21b-4228-a599-51e73f2f921f)

2. Create three EBS volumes of 10 Gib each


![db volume list](https://github.com/user-attachments/assets/dd9c9fcd-463f-4076-b5c5-0fca38ea5a59)


3. Next we connect our instance using the windows terminal






4. We can see the block device currently connected using the command:
    lsblk


![lsblk newly created block](https://github.com/user-attachments/assets/cc3cd401-b39f-411a-ade0-4ead0f82e39c)


    
5. To see all the mounts and free spaces available on the server:
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


        sudo pvcreate /dev/xvdi1

        sudo pvcreate /dev/xvdj1

        sudo pvcreate /dev/xvdk1




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




13. Verify that your Logical Volume has been created successfully by
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

    
13. Configure SELinux Policies: SELinux (Security-Enhanced Linux) is a mandatory access control (MAC) system that provides a robust security framework for Linux systems. It operates by enforcing a set of rules, known as policies, that govern how processes can interact with system resources.


        #Change the ownership of the /var/www/html/wordpress directory and all its contents to the apache user and group.
        sudo chown -R apache:apache /var/www/html/wordpress

        #Change the ownership of the /var/www/html/wordpress directory and all its contents to the apache user and group. -t httpd_sys_rw_content_t sets the context to httpd_sys_rw_content_t, which grants read and write permissions to the httpd service.
        sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R


        #Enable network connectivity for the httpd service
        sudo setsebool -P httpd_can_network_connect=1

    ![configuring SElinux policies output](https://github.com/user-attachments/assets/583d5143-e345-4762-a9ba-4b2f8cef7a49)


**Step 4 — Install MySQL on your DB Server EC2**

Enter the command below to update and install mysql-server:


        sudo yum update
        sudo yum install mysql-server


        
Verify that the service is up and running by using 

        sudo systemctl status mysqld:

![systemctl status mysqld](https://github.com/user-attachments/assets/f9ef0465-f11e-4801-800f-bd2c0e210dc8)

        
We will restart the service and enable it so it will be running even after reboot:


        sudo systemctl restart mysqld
        sudo systemctl enable mysqld

![systemctl restart and enable mysqld](https://github.com/user-attachments/assets/0e1aeb1f-5eb2-4a9f-bf99-19b05ae3725c)



Configure remote access by editing the mysql configuration file with is found at /etc/my.cnf in Mysql installed on Amazon linux 2023. This is equivalent to the 

        sudo nano /etc/my.cnf



 ![bind-address in etc my cnf](https://github.com/user-attachments/assets/39c43d9c-a634-412a-ba05-20af6cfc79ca)


**Step 5 — Configure DB to work with WordPress:**


        #Enter the mysql console
        sudo mysql
        
        #Create a database called 'wordpress'
        CREATE DATABASE wordpress;


        #Create a user named 'myuser' and set the password to 'Password123$'
        CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'Password123$';
        

        #Grant permission to 'myuser' to have access to all information with 'wordpress.'
        GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
        
        #Remove existing cached privileges
        FLUSH PRIVILEGES;

        
        #show databases
        SHOW DATABASES;

        #exit the console
        exit




        
**Step 6 — Configure WordPress to connect to remote database.**



Open MySQL port 3306 on DB Server EC2. For extra security, We shall allow
access to the DB server ONLY from your Web Server's IP address, so in the Inbound Rule
configuration specify source as /32


1. Go to the webserver ec2 and install MySQL client and test that you can connect from your Web Server to your DB server by
 using mysql-client:


        sudo yum install mysql
   
        sudo mysql -u myuser -p -h <DB-Server-Private-IP-address>

   
2. Verify if you can successfully execute SHOW DATABASES; command and see a list of existing
databases.

    ![sql commands on the webserver ec2](https://github.com/user-attachments/assets/a7598382-7ffe-46eb-81db-40f95dbcdf89)

3. Now we have to ensure proper file permissions are set to enable Apache is able to read and write wordpress files.
   We would make apache the owner of the wordpress files by setting directories to 755 permission and by setting files to 644 permission


        sudo chown -R apache:apache /var/www/html/wordpress
        sudo find /var/www/html/wordpress -type d -exec chmod 755 {} \;
        sudo find /var/www/html/wordpress -type f -exec chmod 644 {} \;

   
4. Configure wp-config file to contain remote database details

   Locate the wp-config.php file using the command:

        sudo nano /var/www/html/wordpress/wp-config.php

   Now edit the file by following the format:


        define('DB_NAME', 'your_database_name');
        define('DB_USER', 'your_database_user');
        define('DB_PASSWORD', 'your_database_password');
        define('DB_HOST', 'your_database_server_private_ip');


   ![wp-config php png editing](https://github.com/user-attachments/assets/d3617cd7-2062-4910-8c28-48d30577843a)


       
6. Ensure Apache is configured to serve WordPress. Create a new Apache configuration file



       sudo nano /etc/httpd/conf.d/wordpress.conf # the conf.d folder is used in Redhat distributions unlike ubuntu which uses sites-available

    ![apache config file](https://github.com/user-attachments/assets/d1cc6486-1c68-4c08-a066-9fae28b567ac)

  
8. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from
everywhere 0.0.0.0/0 or from your workstation's IP)


   ![new ib rule for port 80](https://github.com/user-attachments/assets/0832a5ba-353e-4ed1-ac1a-88decbc6a6c1)

9. Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP￾Address>/wordpress/


   ![wordpress site using pub ip](https://github.com/user-attachments/assets/e52bf28c-01c7-42b9-934b-2b80d9a93362)



    ![wordpress pre installation](https://github.com/user-attachments/assets/81c54296-0c0c-4caa-9be6-818120b481f9)


    
    ![allens wordpress site](https://github.com/user-attachments/assets/53eb1023-cf4a-4070-a3a2-25a37b90e3ac)







    Now we have been able to configure Linux storage susbystem and have also deployed a full-scale
    Web Solution using WordPress CMS and MySQL RDBMS!
