# AWS RStudio

Instructions for running R and RStudio Server on an AWS EC2 instance and for creating an RStudio Amazon Machine Image (AMI)

Credit Jagger Villalobos (https://jagg19.github.io/2019/08/aws-r/#long_way)

Prerequisites:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. Create an AWS account  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. Set IAM permissions to allow Amazon EC2 access  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c. Install and configure AWS Command Line Interface (CLI)<br/><br/>  

## 1. Configure a security group that has ports for SSH, HTTP, RStudio<br/>

A security group acts as a virtual firewall for an EC2 instance to control incoming and outgoing traffic.  

Example security group:  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Security group name:** RStudio-security-group  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Description:** Allow SSH, HTTP, RStudio  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**VPC:** &lt;provide vpc ID&gt;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 1 - SSH, Anywhere-IPv4, port 22  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 2 - HTTP, Anywhere-IPv4, port 80  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inbound rule 3 - Custom TCP, Anywhere-IPv4, port 8787 (RStudio)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Outbound rule 1 - All traffic

*Record security group ID  

## 2. Create EC2 instance with an Amazon Linux AMI

Free tier eligible:  
Ubuntu Amazon Machine Image (AMI) Ubuntu Server 18.04 LTS (HVM), SSD Volume Type - ami-0c159d337b331627c (64-bit (x86))  

```
$ aws ec2 run-instances --image-id ami-0c159d337b331627c --count 1 --instance-type t2.micro --key-name EC2 --security-group-ids <security group ID> --subnet-id <subnet ID> --tag-specifications ResourceType=instance,Tags='[{Key=Name,Value=RStudio}]'
```  

Parameters:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--image-id:** 'AMI catalog' in the EC2 portal   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--key-name:** 'Key Pairs' in the EC2 portal  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--security-group-ids:** 'Security Groups' in the EC2 portal   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--subnet-id:** 'Subnets' in VPC portal  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**--tag-specifications:** provides an instance name as the Value, e.g., 'RStudio'

#### Check the instance is running 

```
$ aws ec2 describe-instances --filters "Name=tag:Name,Values=RStudio"
```

*Record instance ID  
*Record PublicDnsName  

### Connect to EC2 instance using Secure Shell Protocol (SSH)  

```
$ ssh -i </path/to/my-key-pair.pem> ubuntu@<my-instance-public-dns-name>
```  
Instructions to generate a key pair: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html  

## 3. Install yum  

Tool for getting, installing, deleting, querying, and managing Linux software packages.  

```
$ sudo apt install yum  
$ sudo yum installed
```  

## 4. Add the The Comprehensive R Archive Network (CRAN) package repository  

Ubuntu repos contain an outdated version of R. The most recent version of RStudio Server can be found here: https://www.rstudio.com/products/rstudio/download-server/    

```
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
$ sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'
```

#### Update Ubuntu package repository  

```
$ sudo apt update
```  

## 5. Install R, R Studio Server and system dependencies  

Base-R (r-base) is the basic software which contains the R programming language.  
r-base-dev is an ubuntu package for compiling R packages and other software depending on R.  

```
$ sudo apt -y install r-base r-base-dev
```  

#### Install debian package manager, gdebi  

GDebi is a package installer for Debian packages on Linux.  

```
$ sudo apt install gdebi-core
```  

#### Install dependencies for R packages  

devtools, tidyverse, sparklyr, RMariaDB  

```
$ sudo apt -y install libcurl4-openssl-dev 
$ sudo apt -y install libssl-dev libxml2-dev libmariadbclient-dev build-essential libcurl4-gnutls-dev
```  

#### Install RStudio Server 

```
$ wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-2022.02.3-492-amd64.deb
$ sudo gdebi -n rstudio-server-2022.02.3-492-amd64.deb
$ sudo rm rstudio-server-2022.02.3-492-amd64.deb
```  

#### Install useful R Packages  

RCurl package is used for reading objects directly from an S3 bucket.  

```
$ sudo R -e "install.packages('RCurl', repos='http://cran.rstudio.com')"
$ sudo R -e "install.packages('devtools', repos='http://cran.rstudio.com')"
$ sudo R -e "install.packages('tidyverse')"
$ sudo R -e "install.packages('RMariaDB')"
```  

## 6. Configure R Studio Server 

#### Set RStudio login credentials  

Add user info to login RStudio 

```
$ sudo adduser rstudio (username = rstudio)
```
Password: rstudio  

Username and password set as 'rstudio' for ease of use.

#### Add RStudio to sudo group  

```
$ sudo usermod -aG sudo rstudio
```  

#### Install Java for RStudio  

Reconfigure the library paths for RStudio use

```
$ sudo apt -y install default-jdk
$ sudo R CMD javareconf
```  

#### Change permissions for R library  

```
$ sudo chmod 777 -R /usr/local/lib/R/site-library
```  

#### Restart RStudio Server  

```
$ sudo rstudio-server restart
```  

## 7. Login to RStudio  

Open a web browser and enter Public DNS(IPv4) as the URL to login to RStudio Server. Specify RStudio port (8787) at the end of the url

&lt;Public DNS(IPv4)&gt;:8787
