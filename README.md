# AWS RStudio

Instructions for running R and RStudio on an AWS EC2 instance and for creating an RStudio Amazon Machine Image (AMI)

Credit Jagger Villalobos (https://jagg19.github.io/2019/08/aws-r/#long_way)

Prerequisites:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. Create an AWS account  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. Set up AWS Command Line Interface (CLI)<br/><br/>  

## 1. Configure a security group that has ports for SSH, HTTP, RStudio<br/>

A security group acts as a virtual firewall for an EC2 instance to control incoming and outgoing traffic.  

Example:  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Security group name:** RStudio-security-group  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Description:** Allow SSH, HTTP, RStudio  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**VPC:** &lt;provide vpc ID&gt;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 1 - SSH, Anywhere-IPv4, port 22  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 2 - HTTP, Anywhere-IPv4, port 80  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 3 - Custom TCP, Anywhere-IPv4, port 8787 (RStudio)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Outbound rule 1 - All traffic

*Record security group ID  

## 2. Create EC2 instance with an Amazon Linux AMI

Free tier eligible: Ubuntu Amazon Machine Image (AMI) Ubuntu Server 18.04 LTS (HVM), SSD Volume Type - ami-0c159d337b331627c (64-bit (x86))  

```
$ aws ec2 run-instances --image-id ami-0c159d337b331627c --count 1 --instance-type t2.micro --key-name EC2 --security-group-ids <security group ID> --subnet-id <subnet ID> --tag-specifications ResourceType=instance,Tags='[{Key=Name,Value=RStudio}]'
```  

Parameters:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--image-id:** 'AMI catalog' in the EC2 portal   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--key-name:** 'Key Pairs' in the EC2 portal  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--security-group-ids:** 'Security Groups' in the EC2 portal   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--subnet-id:** 'Subnets' in VPC portal  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--tag-specifications:** provides an instance name as the Value, e.g., 'RStudio'
