# Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL
# This  double stroke  *||* sysmbols shows command line instruction

## Overview
    In this lab, you create a Cloud Storage bucket and place an image in it. You'll also configure an application running in Compute Engine to use a database managed by Cloud SQL. For this lab, you will configure a web server with PHP, a web development environment that is the basis for popular blogging software. Outside this lab, you will use analogous techniques to configure these packages.

    You also configure the web server to reference the image in the Cloud Storage bucket.

## Objectives
    In this lab, you learn how to perform the following tasks:

   - [x] Create a Cloud Storage bucket and place an image into it.

   - [x] Create a Cloud SQL instance and configure it.

   - [x] Connect to the Cloud SQL instance from a web server.

   - [x] Use the image in the Cloud Storage bucket on a web page.

   

## steps

### Opening your terminal
A authentication your gcloud account_
            || gcloud auth login
B list your projects_
            || glcoud projects list
C setting your project_
            || gcloud config set project qwiklabs-gcp-01-51df6ecbff50


### 1  Deploy a web server VM instance
    
         ||  gcloud compute instances create "my-vm-1" --machine-type "n1-standard-1" --image-project "debian-cloud" --image "debian-9-stretch-v20190213"    --subnet  "default" --tags http             --metadata=startup-script=apt-get\ update$'\n'apt-get\ install\ apache2\ php\ php-mysql\ -y$'\n'service\ apache2\ restart

         ||  gcloud compute --project=qwiklabs-gcp-01-51df6ecbff50 firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

### 2  Create a Cloud Storage bucket using the gsutil command line

######  1 storing my location AND project ID in an environment variable to be used in create my bucket

            || export LOCATION=US

            || export DEVSHELL_PROJECT_ID=qwiklabs-gcp-01-51df6ecbff50

######   2 creating my bucket

            || gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID


######   3  Retrieve a banner image from a publicly accessible Cloud Storage location:

            || gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png

######   4 Copy the banner image to your newly created Cloud Storage bucket:

            || gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

######   5 Modify the Access Control List of the object you just created so that it is readable by everyone:

            || gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

### 3 Create the Cloud SQL instance
    
        || gcloud sql instances create blog-db --password="root" --sql-version=MYSQL_5_7" --zone=us-central1-a

        || gcloud sql databases create host-db --instance="blog-db"

        || gcloud sql users create blogdbuser --instance=blog-db -i blog-db --host='%' --password='root'

_Connect to the Sql instance._

        || gcloud sql instances patch "web front end" --authorized-networks="34.66.179.46/32" 


### 4 Configure an application in a Compute Engine instance to use Cloud SQL

######    1 ssh to bloghost vm to open connection
             || gcloud compute ssh bloghost|<external ip address>
    
######    2 In your ssh session on bloghost, change your working directory to the document root of the web server:
             || cd /var/www/html
    
######    3 Use the nano text editor to edit a file called index.php:
        
             || sudo nano index.php

######    4 Paste the content below into the file:

``` <html>
            <head><title>Welcome to my excellent blog</title></head>
            <body>
            <h1>Welcome to my excellent blog</h1>
            <?php
            $dbserver = "35.195.0.105 ";
            $dbuser = "blogdbuser";
            $dbpassword = "root";
            // In a production blog, we would not store the MySQL
            // password in the document root. Instead, we would store it in a
            // configuration file elsewhere on the web server VM instance.

            $conn = new mysqli($dbserver, $dbuser, $dbpassword);

            if (mysqli_connect_error()) {
                    echo ("Database connection failed: " . mysqli_connect_error());
            } else {
                    echo ("Database connection succeeded.");
            }
            ?>
     </body></html>
```

######  5  Press Ctrl+O, and then press Enter to save your edited file and Press Ctrl+X to exit the nano text editor.


######  6  Restart the web server:

             || sudo service apache2 restart

######  7 Open the  bloghost VM instance's external IP address followed by /index.php. 
     
             || google-chrome   35.228.30.200/index.php
    

### 5  Configure an application in a Compute Engine instance to use a Cloud Storage object


######  1 upload image to my cloud bucket

            || gsutil cp "/path/to/image" gs://$DEVSHELL_PROJECT_ID

######  2   Make the image publicly accessible:

            || gsutil acl ch -u AllUsers:R gs://$DEVSHELL_PROJECT_ID

######  3 Enter this command to set your working directory to the document root of the web server
            ||  cd /var/www/html

######  4 Edit the file with nano text  editor

            || sudo nano index.php

######  5 Use the arrow keys to move the cursor to the line that contains the h1 element. Press Enter to open up a new, blank screen line, and then paste the URL 

         <img src='https://storage.googleapis.com/qwiklabs-gcp-01-51df6ecbff50/myimage' />

         Press Ctrl+O, and then press Enter to save your edited file. and Press Ctrl+X to exit the nano text editor.   

### 7 Restart the web server:  
             || sudo service apache2 restart


