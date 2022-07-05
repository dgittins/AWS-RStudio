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

### Check the instance is running  

```
$ aws ec2 describe-instances --filters "Name=tag:Name,Values=RStudio"
```

*Record instance ID  
*Record PublicDnsName  

## 3. Connect to EC2 instance using SSH  

```
$ ssh -i </path/my-key-pair.pem> ubuntu@<my-instance-public-dns-name>
```  

## 4. Install yum  

Tool for getting, installing, deleting, querying, and managing Linux software packages  

```
$ sudo apt install yum  
$ sudo yum installed
```  

## 5. Add the CRAN repo  

Ubuntu repos contain an outdated version of R  

```
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
$ sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'
```

### Update Ubuntu package repo  

```
$ sudo apt update
```  

## 6. Install R  

```
$ sudo apt -y install r-base r-base-dev
```  

## 7. Install debian package manager, gdebi  

```
$ sudo apt install gdebi-core
```  

## 8. Install dependencies for R packages  

devtools, tidyverse, sparklyr, RMariaDB  

```
$ sudo apt -y install libcurl4-openssl-dev 
$ sudo apt -y install libssl-dev libxml2-dev libmariadbclient-dev build-essential libcurl4-gnutls-dev
```  

## 9. Install RStudio  

```
$ wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-2022.02.3-492-amd64.deb
$ sudo gdebi -n rstudio-server-2022.02.3-492-amd64.deb
$ sudo rm rstudio-server-2022.02.3-492-amd64.deb
```  

## 10. Install useful R Packages  

RCurl used for reading objects directly from S3 bucket  

```
$ sudo R -e "install.packages('RCurl', repos='http://cran.rstudio.com')"
$ sudo R -e "install.packages('devtools', repos='http://cran.rstudio.com')"
$ sudo R -e "install.packages('tidyverse')"
$ sudo R -e "install.packages('RMariaDB')"
```  

## 11. Set RStudio login credentials  

Add user info to login RStudio 

```
$ sudo adduser rstudio (username = rstudio)
```
Password: rstudio  

## 12. Add rstudio to sudo group  

```
$ sudo usermod -aG sudo rstudio
```  

## 13. Install Java for RStudio  

Reconfigure the library paths for RStudio use

```
$ sudo apt -y install default-jdk
$ sudo R CMD javareconf
```  

## 14. Change permissions for R library  

```
$ sudo chmod 777 -R /usr/local/lib/R/site-library
```  

### Restart rstudio-server  

```
$ sudo rstudio-server restart
```
