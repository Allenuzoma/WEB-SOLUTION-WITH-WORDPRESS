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




5
