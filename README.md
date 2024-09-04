# Hosting and Running a multi tier web application on AWS Cloud 
## Contents
* [Introduction](#Introduction "Goto Introduction")
* [AWS Services Used](#AWS-Services-Used "Goto AWS Services Used")

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

    
