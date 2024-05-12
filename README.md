# AWS-Re-Architecting-Web-App-on-AWS

AWS Refactor with AWS Beanstalk
TOOLS:
EC2 instances, ELB, Autoscaling, EFS/S3, RDS, ELASTIC CACHE, ACTIVE MQ, ROUTE 53, CLOUDFRONT

RDS

vprofile-rds-mysql.cxjfmdznrqoy.us-east-2.rds.amazonaws.com

Master username - admin
Master password - KgHMKVmLA597HnSvF9MQ


1. First create an RDS, and before thar create an Subnets group and select all the subnets in the region
2. Create a Parameter Group because we cant ssh into the RDS or change the configuration, so we need Parameter Group for that. So if you need to make any configuration changes in the RDS, you can do it through the Parameter Group.
3. Now we can create the Database, I created MySQL db.

Amazon ElastiCache

vprofile-elasticache-svc.afjeve.cfg.use2.cache.amazonaws.com

1. Similar to RDS, firstly, create the Subnets and then create the Parameter Groups.
2. Create the Elastic Cache 

AMAZON MQ/ RABBITMQ

b-fa09774a-c045-4a3d-a13e-04e37dd3bb70.mq.us-east-2.amazonaws.com

RabbitMQ access
username - rabbit
password - Blue7822bunny

1. Create the Broker engine and select your configuration.


DB Iniatialization.

1. The only way to access our DB( since it can only be access privately) is to create an EC2 instance in the same vpc, so you can connect our RDS and query our DB.
2. Create a security group for the Mysql-client instance to allow connect from your IP(or your organization IP)
3. Then add the allow your vprofile-backend-SG to allow connect from the Security Group of the MySQL that was just created.
2. After ssh into the MySQL instance, apt update and install mysql-client if its ubuntu engine. Confirm if MySQL is installed.
4. Clone the repo from GitHub into the instance, cd into the VPROFILE PROJECT and run the database query into source code into the MySQL instance. (ubuntu@ip-172-31-43-134:~/vprofile-project$ mysql -h vprofile-rds-mysql.cxjfmdznrqoy.us-east-2.rds.amazonaws.com -P 3306 -u admin -pKgHMKVmLA597HnSvF9MQ accounts < src/main/resources/db_backup.sql)
5.Show Tables, then you find that DB account created.
6. Now you can terminate the EC@ intshace that was created.
7. Make sure you copy and save the endpoint of RabbitMQ and Elastic Cache somewhere safe.

AMAZON ELASTIC BEANSTALK	

Elastic Beanstalk is a service for deploying and scaling web applications and services. Upload your code and Elastic Beanstalk automatically handles the deployment—from capacity provisioning, load balancing, and auto scaling to application health monitoring.

1. Before you start creating Beanstalk, you need to create a ROLE firstly due to recent issue with BEANSTALK.
2. Create the Amazon Beanstalk and configure according...

UPDATING CONFIGRATION. 
1. Update the ACL in the S3 bucket that was created by the Amazon Elastic Beanstalk.
2. Under PERMISSION, update the Object Ownership so your AWS accounts can have access to the s3 bucket.
3. Update the PROCESSES in BeanStalk/Environment/Processes and configure the health check to /login as per the application requirement.
4. Add https listener. Add port 443 https listener and select SSL certificate from ACM 
5. Update the security group of the EC2 instances created by Beanstalk. Copy the SG ID of one of the EC2 instance created by Beanstalk, and configure the Inbound rule of your Backend-service to "Allow all traffic from Beanstalk Ec2 instance SG". This allow RDS, Memcached and RabbitMQ to connect to the Beanstalk

BUILD AND DEPLOY ARTIFACTS 
1. Git clone the application form the repo to your local machine
2. Update the Application Properties with the Endpoints, Username, Password and maybe PORT of RDS, Elastic Cache and RabbitMQ in the application file and save.
3. Open Bash terminal, check to see if "mvn" is install on your machine. Because that's what needed to build the Application Artifact from your application file.
4.Run "mvn install" to build your application artifact.
5.After the Application Artifact is been built, Open Beanstalk, Upload and deploy the Artifact "vprofile-v2.war" 
6. After the Deployment, the website is not accessible but it's not secured yet because the beanstalk is point to the ELB and not the Domain.
7. Now it's time to update the Domain DNS. Add a new record, copy the endpoint of the Beanstalk and paste in the NEW RECORD, save and give few mins. Now you should be able to access the wbsite with your domain name and secured as well. 
8. Login with 
username - admin_vp
password - admin_vp  If you login successful, then your db is successful attached.
9. Now confirm if RabbitMQ and Memcached is working fine as well.

CLOUDFRONT
Amazon CloudFront is a fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency and high transfer speeds.

1. Create your CloudFront by selecting the ELB created by the Beanstalk.
3. After creating the CloudFront, copy the Distribution Domain Name or Endpoint, and use it to create a new record on your Domain DNS 
