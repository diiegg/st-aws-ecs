#  Set up a production environment for docker on AWS ECS.

## Focus

I will show how to create a production environment to run docker applications


**Prerequisites:**

Previous docker synfony app
AWS account
AWS CI configured
ECF stack 
Production database server
Cloud formation template


#  Pushing the images into a repository

Log in into ECR and create a new repository to host the docker images we are using the eu-west-1 region

    aws ecr get-login-password \
        --region eu.west.i \
    | docker login \
        --username AWS \
        --password-stdin 631442776722.dkr.eu-west-1.amazonaws.com

**Create the repository**

    aws ecr create-repository --repository-name cosmic-coyote --region eu-west-1

  **Tag the images using docker command to the AWS specifications**


    docker tag image-id aws_account_id.dkr.ecr.region.amazonaws.com/my-web-app

**Look like something like this tag the php image, do the same for the nginx image**

    docker tag 82539d56ce70 <aws-id>.dkr.eu-west-1.amazonaws.com/sf-php:latest
    docker tag 82539d56234 <aws-id>.dkr.eu-west-1.amazonaws.com/sf-nginx:latest

**Now that you have both images ready php and nginx push them into the repository we created.**

    docker push 82539d56ce70 <aws-id>.dkr.eu-west-1.amazonaws.com/sf-php:latest
    docker push 82539d56234 <aws-id>.dkr.eu-west-1.amazonaws.com/sf-nginx:latest

**Log in into AWS console and navigate into the ECR to see the images**

![enter image description here](https://user-images.githubusercontent.com/12648295/104719085-d8efd400-5723-11eb-972f-d12909b0b9aa.png)

**Inside the repository**

![enter image description here](https://user-images.githubusercontent.com/12648295/104719191-05a3eb80-5724-11eb-8377-183c3ea98676.png)

## Setting up the Infrastructure
    
We will need a production database, set up a production database:

[MySQL](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)
[Postgres](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-20-04)
[MariaDB](https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-ubuntu-20-04)

From this Link choose one of the 2 templates to install the ECS cluster to host our application,  and go to the cloud formation service choose to create stack-->new resources, select *Upload a template file* if you have the template store in your machine or Amazon S3 URL if is store in an *AWS s3 bucket* then next and follow the instructions.

![enter image description here](https://user-images.githubusercontent.com/12648295/104722040-8fa08400-5725-11eb-9013-a7446bac3416.png)

**After some minutes your stack should be ready and running.**

![enter image description here](https://user-images.githubusercontent.com/12648295/104722532-ddb58780-5725-11eb-9f07-c5f9e19db4c5.png)

**Go to  ec2 instances an ECS cluster should be ready to host the service**

![enter image description here](https://user-images.githubusercontent.com/12648295/104722653-0e95bc80-5726-11eb-90c1-8be146d7bc7b.png)

## Deploy the service into the new cluster

We need to define a task definition for each container we are running, in this case the PHP an NGinX run together.

We have to option to set up the task definition

1 Using AWS cloud formation we can set u the task definition with a yml file

2 Using the graphic interface

option 2 :

Go to ECS and choose task definition --> Create new task definition 

![enter image description here](https://user-images.githubusercontent.com/12648295/104722653-0e95bc80-5726-11eb-90c1-8be146d7bc7b.png)

Select ec2 and next

![enter image description here](https://user-images.githubusercontent.com/12648295/104724042-0b9bcb80-5728-11eb-9871-eea10cf4e97a.png)

Select a Name for the task definition, network select *host*, add the memory and CPU for the task and add container


![enter image description here](https://user-images.githubusercontent.com/12648295/104724255-56b5de80-5728-11eb-9c8e-ee0fdf519329.png)

**In the container section choose a name, the URL of the docker image we are using, in this case the URL of the repository we create, and port mapping 80 - 80 click add.**

- Select add container again and add the php container

We are setting up a task definition with 2 containers in the same one because PHP need the nginx as a web server

**You will have something like this at the end** 

![enter image description here](https://user-images.githubusercontent.com/12648295/104724767-09863c80-5729-11eb-9d01-5ea2c57967e4.png)

**The service should be ready and running**

![enter image description here](https://user-images.githubusercontent.com/12648295/104724886-39354480-5729-11eb-8256-167fde095908.png)

**Select Cluster and select the cluster you create before and your service should be ready and running**


![enter image description here](https://user-images.githubusercontent.com/12648295/104725033-6f72c400-5729-11eb-939a-7551a391515a.png)


**Check that the service is running with the public IP of the cluster**

![enter image description here](https://user-images.githubusercontent.com/12648295/104725204-a8129d80-5729-11eb-9ba1-5e33fd5c3f6f.png)



Reference

[Docker on ECS](https://docs.docker.com/cloud/ecs-integration/)
