#  AWS cloud formation templates

##  The following template create ECS cluster using ec2 instances



#  ecs-master.template

This Template creates one or more ec2 instances and an ECB with a load balancer facing the internet, the service have a public IP address, so it can initiate direct communication with other services on the internet.

This template doesn't include a service deploy.



![enter image description here](https://user-images.githubusercontent.com/12648295/104727656-9f23cb00-572d-11eb-9f94-7fda3ffe5839.png)

# service.template

- Deploy a service into an ECS cluster behind the load balancer

Reference:

[Cloud formation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-service.html)
