# Hosting and Running a Multi Tier Web Application on AWS Cloud 
## Contents
* [Introduction](#Introduction "Goto Introduction")
* [AWS Services Used](#AWS-Services-Used "Goto AWS Services Used")
* [Security Groups](#Security-Groups "Goto Security Groups")
* [EC2 Instances](#EC2-Instances "Goto EC2 Instances") 
* [Hosted Zones(Amazon Route53)](#Hosting-Zones "Goto Hosting Zones(Amazon Route53)")
* [Building and Deploying Artifacts](#Building-and-Deploying-Artifacts "Goto Building and Deploying Artifacts")
* [Load Balancer and DNS](#Load-Balancer-and-DNS "Goto Load Balancer and DNS") 

## Introduction
Using lift and shift strategy, rearchitecting on-premises application elements, leveraging cloud services and optimizations that provide the most significant benefits.
#### Disadvantages of on-premises 
* Complex Management
* Scalability Challenges
* UpFront Capital Expenses and regular Operational Expenses
* Manuel processes, difficult to automate, and time consuming
#### Advantages of cloud
* Ease of Infra management
* Easy scaling up/down
* No upfront expensed, pay-as-you-go system
* Using Infrastructure as a Service
* Easy automation

## AWS Services Used
### Services
* EC2 Instances- VMs for Tomcat, RabbitMQ, Memcache, MySQL
* Elastic Load Balancer- NGINX load balance replacement
* Autoscaling- Automation for VM scaling
* S3/EFS Storage- Shared Storage
* Route 53- Private DNS Service
### Architecture
![image](https://github.com/user-attachments/assets/f997a0a9-713f-4250-be6e-8583d54bcc48)
### Flow of execution
1. Login to AWS Account
2. Create Key Pairs
3. Create Security Group
4. Launch Instances with user data
5. Update IP to name mapping in Route53
6. Build application from source code
7. Upload to S3 bucket
8. Download artifact to Tomcat EC2 instace
9. Setup ELB with HTTPS(Certification from Amazon Certification Manager)
10. Map ELB endpoint to website name in GoDaddy DNS
11. Verify
12. Build Autoscaling group for Tomcat Instance

## Security Groups
#### For Elastic Load Balancer(ELB)
![image](https://github.com/user-attachments/assets/ff16a43d-b818-4827-a38b-b71e46ec6081)
#### For Tomcat Instance
Source- From security group of ELB
![image](https://github.com/user-attachments/assets/2293a9a9-fb93-4088-9ce8-9d5716d0a31b)
#### For Backend Srvices
Source- From secrity group of Application(Tomcat)

Using default IPs for memecache and rabbitMQ
![image](https://github.com/user-attachments/assets/33154bb3-40c9-4135-8239-dfa32c0534d8)
After creating this SG, making a change in inbound rules(For all these services because they will interact to each other also)
![image](https://github.com/user-attachments/assets/e7a6f5ab-e294-4668-99eb-1b2bf6f7a966)
Allowing access from my IP
![image](https://github.com/user-attachments/assets/42fa8a2a-f3c5-4f60-9aa4-93935315b583)

## EC2 Instances
Using scripts in `/userdata` directory
![image](https://github.com/user-attachments/assets/4da975c1-5e53-46ac-8517-ffacdbb5d3ce)
Checking each instance by logging in using-

```
Check connect tab before this:
ssh -i "web-key.pem" ec2-user@public-ip-of-instance
e.g. ssh -i /d/Users/Vinayak/Downloads/web-key.pem ec2-user@54.90.94.104

systemctl status service-name

Or we can use:
ss -tunlp | grep default-port
e.g. for memcached: ss -tunlp | grep 11211 
```
For MySQL, checking tables-
```
mysql -u admin -padmin123 accounts
show tables;
```

## Hosted Zones(Amazon Route53)
A hosted zone is a container for records, and records contain information about how you want to route traffic for a specific domain, such as example.com, and its subdomains (acme.example.com, zenith.example.com). A hosted zone and the corresponding domain have the same name. There are two types of hosted zones:
* Public hosted zones contain records that specify how you want to route traffic on the internet.
* Private hosted zones contain records that specify how you want to route traffic in an Amazon VPC.
![image](https://github.com/user-attachments/assets/67e6585d-1ea1-47ff-a331-f9be86615da8)
### Creating records
After you create a hosted zone for your domain, such as example.com, you create records to tell the Domain Name System (DNS) how you want traffic to be routed for that domain.
For example, you might create records that cause DNS to do the following:
* Route internet traffic for example.com to the IP address of a host in your data center.
* Route email for that domain (ichiro@example.com) to a mail server (mail.example.com).
* Route traffic for a subdomain called operations.tokyo.example.com to the IP address of a different host.
#### For db01
Value- Private IP of db01 instance
![image](https://github.com/user-attachments/assets/b6dd1a66-a52f-4a95-86d4-e4321a30e3e9)
Similarly, making records for other services.
![image](https://github.com/user-attachments/assets/6863a0db-44dc-4589-973e-5710218335d7)

## Building and Deploying Artifacts
Creating artifact in our local machine and then uploading it to S3 bucket. Before that we have to see our `application.properties` file in `/src/main/resources` where we have to check host-
![image](https://github.com/user-attachments/assets/b084de14-1b9a-4c12-9e24-67435d85958f)

After that to create artifact-
```
mvn install
```
After running this command buiding will start and a new forlder `/target` will be created. Now will push this to a S3 bucket. For that we need a IAM user with that kind of permissions.
#### Creating an IAM user
![image](https://github.com/user-attachments/assets/8ab869f5-7ea5-4e37-b97a-23075287e633)
We are going to use this user's access key to access the S3 bucket. Creating access key-
![image](https://github.com/user-attachments/assets/9f507c64-1e62-46a6-9c40-6a8f1f70f5bf)
Configuring the access key-
```
In git basg-
aws configure
```

#### Creating a S3 bucket
```
From the same folder where your artifact file is present-
aws s3 mb s3://unique-bucket-name
For copying files to S3 bucket-
aws s3 cp target/vprofile-v2.war s3://bucket-name-that-you-made
```
![image](https://github.com/user-attachments/assets/ad523766-50a2-4316-b7d2-b0ec563c5f40)

#### Deploying bucket
Making a new IAM role for our App Instance, giving it full S3 access.
![image](https://github.com/user-attachments/assets/78d34431-7090-4e5d-b2f7-49aab9e70866)
Then, modifying IAM role for our App Instance-
![image](https://github.com/user-attachments/assets/374cfc7a-3ead-47fb-945d-f27dfcab5754)


Logging in to our app instance and installing AWS CLI-
```
apt install awscli -y
```
Testing if our IAM Role is attached or not-
```
aws s3 ls      \\It should list all S3 buckets
```
Then downloading our artifact in tmp file from S3 bucket-
```
aws s3 cp s3://vinayak-project-artifact/vprofile-v2.war /tmp/
```
![image](https://github.com/user-attachments/assets/2c02baa9-d027-4c18-bca2-cd0fe9b160b8)
then-
```
systemctl stop tomcat9
rm -rf /var/lib/tomcat9/webapps/ROOT   \\Removing any file present in webapps folder
cp /tmp/vprofile-v2.war /var/lib/tomcat9/webapps/ROOT.war    \\Copying our artifact file to tomcat folder
systemctl start tomcat9   \\After this command executes, a new ROOT folder will be created extracting ROOT.war
```

To check if our website is up and running-
```
public-IP-of-app-instance:8080
eg. 44.207.3.216:8080
```

## Load Balancer and DNS
#### Creating Target Group
Choose target type as Instance, name the target group, provide the port no. where app is running and health check path which is `/login` in my case, and we have to choose override option in advanced health settings and change the port no. to 8080 also. Then select the instance and provide port no. there also and then click create target group.
![image](https://github.com/user-attachments/assets/e89b28e0-308c-45d0-9a26-8e26da86eba3)

#### Creating Load Balancer
Load Balancer will route the traffic to this target group.
![image](https://github.com/user-attachments/assets/3608f58c-d4e7-4633-8178-adb180644d67)

Now, copying the DNS name and creating a new DNS record in our website DNS management-
![image](https://github.com/user-attachments/assets/797a31c5-1943-4d5c-899f-94c5aa0f9cb1)
 
After waiting 10-15 mins, website will be up and running-
![image](https://github.com/user-attachments/assets/3a365c25-802d-4121-b39c-49405cdbe85f)

We can login to webiste(User-admin_vp password-admin_vp)
![image](https://github.com/user-attachments/assets/60360fa4-dbe9-44bb-bfd4-6fb9b80f4661)

Checking our services working or not-
![image](https://github.com/user-attachments/assets/f0802aa0-5aba-47fb-8c2a-00d5eccb7a9b)
![image](https://github.com/user-attachments/assets/4b81e5e4-2d9e-4dd5-b85f-76724bc779c9)

















    
