Ultimately the goals of this project are to have:
 - a free, sustainable webserver, running in the cloud
 - secure, https only site,
 - allow multiple domains to be hosted
 - using static IP's (free) and auto-update of IP's
 - as automated as possible (not have to use AWS console)
	- use AWS CLI and/or terraform
 - limited access IAM user roles

Steps
 - use a free tier AWS account
 - register domain name via Route53 and create a new hosted zone
 - create a new VPC using AWS best practices
	https://aws.amazon.com/answers/networking/aws-single-vpc-design/
	You will likely only need a handfull of IP addresses. 
	I prefer a Network such as 10.1.1.0/24, Netmask 255.255.255.0, Range 10.1.1.1-10.1.1.254
 - create a new Subnet in the VPC
	Be sure to assign the correct VPC to the Subnet 
	The subnet can be from the size of the VPC at /24 up to /28
	I prefer Subnet CIDR block of 10.1.1.0/27, Range 10.1.1.1-10.1.1.30 (call it Public)
		 Subnet CIDR block of 10.1.1.32/27, Range 10.1.1.33-10.1.1.63 (call if Private)
 - set your Subnet to "Enable auto-assign Public IP"
 - create a Security Group for your VPC
 - create inbound rules such as: HTTP (80), HTTPS (443), SSH (22)

 - create a new Ubuntu EC2 instance
 - download certbot

 - configure nginx.conf files to registered domain
 - create a link in your home folder to get to 

 - setup a bash script to automatically run upon instance load to update the DNS record to the correct public IP address
	https://github.com/nealalan/update_route53
	This will allow you to ssh into your instance using the domain name instead of the newly assigned public IP address


 - To be continued...
