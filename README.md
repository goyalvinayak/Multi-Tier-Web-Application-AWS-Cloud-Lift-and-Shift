# Hosting and Running a Multi Tier Web Application on AWS Cloud 
## Contents
* [Introduction](#Introduction "Goto Introduction")
* [AWS Services Used](#AWS-Services-Used "Goto AWS Services Used")
* [Security Groups](#Security-Groups "Goto Security Groups")
* [EC2 Instances](#EC2-Instances "Goto EC2 Instances")

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

## Building and Deploying Artifacts
#### Creating a S3 bucket






    
