# AWS RStudio

Workflow for running R and RStudio Server on an AWS EC2 instance

Prerequisites:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. Create an [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. Set [IAM permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html) to allow Amazon EC2 access  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c. Install and configure [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) (CLI)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d. Create and configure an [Amazon Virtual Private Cloud](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/gsg_create_vpc.html) (Amazon VPC)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;e. Create an [Amazon EC2 key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)<br/><br/>  

## 1. Configure a security group that has ports for SSH, HTTP, RStudio<br/>

A security group acts as a virtual firewall for an EC2 instance to control incoming and outgoing traffic. Security groups can be created using the [Amazon VPC console](https://console.aws.amazon.com/vpc/) or using the [AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html).  

Example security group:  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Security group name:** RStudio-security-group  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Description:** Allow SSH, HTTP, RStudio  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**VPC:** &lt;vpc ID&gt;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 1 - SSH, Anywhere-IPv4, port 22  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 2 - HTTP, Anywhere-IPv4, port 80  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 3 - Custom TCP, Anywhere-IPv4, port 8787 (RStudio)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Outbound rule 1 - All traffic  

*Record security group ID<br/><br/>

## 2. Create EC2 instance with an Amazon Linux AMI  

Amazon EC2 provides a wide selection of instance types optimized for different uses. [General purpose instances](https://aws.amazon.com/ec2/instance-types/) provide a balance of compute, memory and networking resources.  

An Amazon Machine Image (AMI) is a basic configuration that serves as a template for an EC2 instance. 

Free tier availability instance:  
t2.micro - 1 vCPU, 1.0 RAM (GiB)

Free tier eligible AMI:  
Ubuntu Amazon Machine Image (AMI) Ubuntu Server 18.04 LTS (HVM), SSD Volume Type - ami-023fc89db93991d87 (64-bit (x86))  

#### Launch instance via CLI

```
$ aws ec2 run-instances --image-id ami-023fc89db93991d87 --count 1 --instance-type t2.micro --key-name <key pair name> --security-group-ids <security group ID> --subnet-id <subnet ID> --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=RStudio}]' #name the instance 'RStudio'
```  

Parameters:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--image-id:** 'AMI catalog' in the EC2 portal   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--key-name:** 'Key Pairs' in the EC2 portal  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--security-group-ids:** 'Security Groups' in the EC2 portal   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--subnet-id:** 'Subnets' in VPC portal  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--tag-specifications:** provide an instance name, e.g., 'RStudio'

#### Check the instance is running 

```
$ aws ec2 describe-instances --filters "Name=tag:Name,Values=RStudio"
```

*Record instance ID  
*Record PublicDnsName  

#### Connect to EC2 instance using Secure Shell Protocol (SSH)  

```
$ ssh -i </path/to/my-key-pair.pem> ubuntu@<my-instance-public-dns-name>
```  

Default usernames for different AMIs are listed here: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connection-prereqs.html  

## 3. Install yum  

Tool for getting, installing, deleting, querying, and managing Linux software packages.  

```
$ sudo apt install yum  
$ sudo yum installed
```  

## 4. Add the The Comprehensive R Archive Network (CRAN) package repository  

Ubuntu repositories contain an outdated version of R. The most recent version of RStudio Server can be found here: https://www.rstudio.com/products/rstudio/download-server/ and the command below updated accordingly.    

```
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
$ sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'
```

#### Update Ubuntu package repository  

```
$ sudo apt update
```  

## 5. Install R and system dependencies  

r-base is the basic software which contains the R programming language. r-base-dev is an ubuntu package for compiling R packages and other software depending on R.  

```
$ sudo apt -y install r-base r-base-dev
```  

#### Install R packages  

devtools, tidyverse, sparklyr, RMariaDB  

Dependencies:  

```
$ sudo apt -y install libcurl4-openssl-dev 
$ sudo apt -y install libssl-dev libxml2-dev libmariadbclient-dev build-essential libcurl4-gnutls-dev
```    

R packages:

```
$ sudo R -e "install.packages('RCurl', repos='http://cran.rstudio.com')"
$ sudo R -e "install.packages('devtools', repos='http://cran.rstudio.com')"
$ sudo R -e "install.packages('tidyverse')"
$ sudo R -e "install.packages('RMariaDB')"
```  

## 6. Install and configure R Studio Server

#### Install debian package manager, gdebi  

GDebi is a package installer for Debian packages on Linux.  

```
$ sudo apt install gdebi-core
```  

#### Install RStudio Server 

```
$ wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-2022.02.3-492-amd64.deb
$ sudo gdebi -n rstudio-server-2022.02.3-492-amd64.deb
$ sudo rm rstudio-server-2022.02.3-492-amd64.deb
```   

#### Set RStudio login credentials  

Add user information to login to RStudio 

```
$ sudo adduser rstudio (username = rstudio)
```
Password: rstudio  

Username and password set as 'rstudio' for ease of use.<br/><br/>  

#### Add RStudio to sudo group  

```
$ sudo usermod -aG sudo rstudio
```  

#### Install Java for RStudio  

Reconfigure the library paths for RStudio use:

```
$ sudo apt -y install default-jdk
$ sudo R CMD javareconf
```  

#### Change the access permissions for R library  

```
$ sudo chmod 777 -R /usr/local/lib/R/site-library
```  

#### Restart RStudio Server  

```
$ sudo rstudio-server restart
```  

## 7. Access RStudio Server  

Open a web browser and enter Public DNS(IPv4) followed by the RStudio port (8787) as the URL:

&lt;Public DNS(IPv4)&gt;:8787  

*Use the credentials (rstudio) created earlier in the workflow.<br/><br/>

## 8. Create an AMI from the EC2 Instance  

To save the installed programs and settings, an AMI can be created using the [EC2 portal](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/tkv-create-ami-from-instance.html) or the [AWS CLI](https://awscli.amazonaws.com/v2/documentation/api/2.0.34/reference/ec2/create-image.html)

Example AMI:  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Name:** RStudio-Ubuntu-Server-18.04-LTS-(HVM)-SSD-Volume  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Description:** RStudio Ubuntu Server 18.04 LTS (HVM), SSD Volume

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Delete on termination:** Disable (EBS volume will not be deleted on termination of the EC2 instance)  

*Record AMI ID for future use
