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

#### Steps

##### 1 Create the VM

######  1 Set account to be used for the projects

            || gcloud config set account student-00-1071551d8724@qwiklabs.net

######  2 Create a project

            || exports MY_PROJECT=qwiklabs-gcp-04-75b71598f1d9
            || gcloud projects create $MY_PROJECT

######  3 set the projects 
            || gcloud config set $MY_PROJECT

######  4 Creating the VM instances 

       




gcloud beta compute --project=qwiklabs-gcp-04-75b71598f1d9 instances create mc-server --zone=us-central1-a --machine-type=e2-medium --subnet=default --address=104.197.181.73 --network-tier=PREMIUM --maintenance-policy=MIGRATE  --metadata=startup-script-url=https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh  --service-account=446333601954-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/trace.append,https://www.googleapis.com/auth/devstorage.read_write --tags=minecraft-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mc-server --create-disk=mode=rw,size=50,type=projects/qwiklabs-gcp-04-75b71598f1d9/zones/us-central1-a/diskTypes/pd-ssd,name=minecraft-disk,device-name=minecraft-disk --reservation-affinity=any


firewall 
gcloud compute --project=qwiklabs-gcp-04-75b71598f1d9 firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server