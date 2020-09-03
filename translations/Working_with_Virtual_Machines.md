# Working with Virtual Machines

## the symbol (||) show the command line instruction

## Overview
    In this lab, you set up a game application—a Minecraft server.

    The Minecraft server software will run on a Compute Engine instance.

    You use an n1-standard-1 machine type that includes a 10-GB boot disk, 1 virtual CPU (vCPU), and 3.75 GB of RAM. This machine type runs Debian Linux by default.

    To make sure there is plenty of room for the Minecraft server's world data, you also attach a high-performance 50-GB persistent solid-state drive (SSD) to the instance. This dedicated Minecraft server can support up to 50 players.

### Objectives
    In this lab, you learn how to perform the following tasks:

    Customize an application server

    Install and configure necessary software

    Configure network access

    Schedule regular backups

### Steps

#### 1 Create the VM

######  1 Set account to be used for the projects

            || gcloud config set account student-00-1071551d8724@qwiklabs.net

######  2 Create a project

            || exports MY_PROJECT=qwiklabs-gcp-04-75b71598f1d9
            || gcloud projects create $MY_PROJECT

######  3 set the projects 
            || gcloud config set $MY_PROJECT

######  4 Creating the VM instances 

            || gcloud compute instances create mc-server --zone=us-central1-a --machine-type=e2-medium --subnet=default --address=104.197.187.73  --metadata=startup-script-url=https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh,shutdown-script-url= https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh --tags=minecraft-server --image=debian-9-stretch-v20200902  --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mc-server --create-disk=mode=rw,size=50,type=projects/qwiklabs-gcp-04-75b71598f1d9/zones/us-central1-a/diskTypes/pd-ssd,name=minecraft-disk,device-name=minecraft-disk

#### 2 Prepare the data disk

###### 1 connecting to mc-server

            || gcloud compute ssh mc-server --zone us-central1-a

###### 2 To create a directory that serves as the mount point for the data disk

            || sudo mkdir -p /home/minecraft

###### 3 To format the disk

            || sudo mkfs.ext4 -F -E lazy_itable_init=0,
                lazy_journal_init=0,discard \
                /dev/disk/by-id/google-minecraft-disk

###### 4 To mount the disk, run the following command:

            || sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft

#### 3 Install and run the application

   *Install the Java Runtime Environment (JRE) and the Minecraft server*

###### 1 In the SSH terminal for mc-server, to update the Debian repositories on the VM, run the following command:

            || sudo apt-get update
###### 2 After the repositories are updated, to install the headless JRE, run the following command:

            || sudo apt-get install -y default-jre-headless
###### 3 To navigate to the directory where the persistent disk is mounted, run the following command:

            || cd /home/minecraft
###### 4 To install wget, run the following command:

            || sudo apt-get install wget
                If prompted to continue, type Y

###### 5 To download the current Minecraft server JAR file (1.11.2 JAR), run the following command:

            || sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar

###### 6 Initialize the Minecraft server

            || sudo java -Xmx1024M -Xms1024M -jar server.jar nogui

                The Minecraft server won't run unless you accept the terms of the End User Licensing Agreement (EULA).

            || nano eula.txt
                Change the last line of the file from eula=false to eula=true

   _Create a virtual terminal screen to start the Minecraft server_
      
      <!-- If you start the Minecraft server again now, it is tied to the life of your SSH session: that is, if you close your SSH terminal, the server is also terminated. To avoid this issue, you can use screen, an application that allows you to create a virtual terminal that can be "detached," becoming a background process, or "reattached," becoming a foreground process. When a virtual terminal is detached to the background, it will run whether you are logged in or not. -->

###### 7 to install screen

            || sudo apt-get install -y screen

###### 8 to start the mc-server in screen terminal 
            || sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui

###### 9 To detach the screen terminal, press Ctrl+A, Ctrl+D. The terminal continues to run in the background. To reattach the terminal, run the following command:

            || sudo screen -r mcs

##### 4 Allow client traffic

            || gcloud compute --project=qwiklabs-gcp-04-75b71598f1d9 firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server

##### 5 Schedule regular backups

######  1 create a storage bucket 

            || export MY_BUCKET=qwiklabs-gcp-04-75b71598f1d9

            || gsutil mb gs://$MY_BUCKET-minecraft-backup

######  2 create backup script 

            || sudo nano /home/minecraft/backup.sh
    
######  3  add the following line  inside backup.sh

            #!/bin/bash
            screen -r mcs -X stuff '/save-all\n/save-off\n'
            /usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
            screen -r mcs -X stuff '/save-on\n'

######  4 Press Ctrl+O, ENTER to save the file, and press Ctrl+X to exit nano

######  5 To make the script executable

            || sudo chmod 755 /home/minecraft/backup.sh


###### Test the backup script and schedule a cron job

###### 6 In the mc-server SSH terminal, run the backup script:

            || . /home/minecraft/backup.sh

###### 7 you can schedule a cron job to automate the task. by opening the cron table for editing

            || sudo crontab -e

###### 8 select nano when prompted and At the bottom of the cron table, write the following line:


            || 0 */4 * * * /home/minecraft/backup.sh


 *Press Ctrl+O, ENTER to save the cron table, and press Ctrl+X to exit nano*

##### 6  Server maintenance

             ||  1 sudo screen -r -X stuff '/stop\n'



- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item









            







