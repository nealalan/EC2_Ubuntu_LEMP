## [nealalan.github.io](https://nealalan.github.io)/[EC2_Ubuntu_LEMP](https://nealalan.github.io/EC2_Ubuntu_LEMP)

## Project goals
- Create and install a free, sustainable webserver, running in the cloud.
- Configure the server to be secure, using https (SSL/TLS)
- Flexibility and scalibility to allow multiple domains to be hosted
- Leverage static IP addresses (free in AWS free tier)
- Consider as much automation as possible (using the command line or scripts remotely, instead of logging into the AWS console)
- [Principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) using AWS [IAM](https://aws.amazon.com/iam/) security controls.

![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/banner_lemp1-1.png)
## Terminology
- LEMP server : Software and packages installed including: Linux, Nginx, MySQL, PHP (LEMP stack)
- AWS [EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) : Essentially a server running in the cloud, that you have total control over. you create it, start it, configure it and kill it as you please. And you pay for it if you use too much.
- [AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) : Amazon Machine Image is an image of an operating system that is loaded when you create an AWS EC2 instance. Many of these are free.
- [Ubuntu](https://en.wikipedia.org/wiki/Ubuntu_(operating_system)) Linux : open source desktop, laptop and server grade operating system.
- Other Linux / AWS Linux AMI choices : Ubuntu is so widely used and supported - it's the choice of most. I stick with it for reasons you can also read here, [What type of AMI should you use?](https://www.brandorr.com/blog/what-type-of-ami-should-you-use)
- SSL/[TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security)/Secure Site : Assuming you use a new, commonly used browser, when you see https:// and a little padlock that is locked next to the address, you are being indicated the site is a secure site. If you are doing anything other than reading a site, it should have these. Never enter any data into an http:// site (without the "s") because this is not secure. The information is sent over the internet as text. If it's your password or a credit card number, it's being exposed. (Note: SSL should no longer be used over the internet. TLS is its replacement.)

## Lets get started...
What I won't go over: 
- You will need to have the ability to use the command line. If you're on a windows machine and want some practice, you might try doing it from your web browser from Google [Cloud Console](https://console.cloud.google.com) for free.
- You will want to have an understanding of getting around the folder structure.
- PASSWORD MANAGER!!!! If you're not using a password manager, you might not be ready for cloud technology. Here's a good article on [Consumer Reports: Everything You Need to Know About Password Managers](https://www.consumerreports.org/digital-security/everything-you-need-to-know-about-password-managers/)
- I use [LastPass](https://www.lastpass.com/) and have been for years. I think it's the best and it lets you put in all sorts of things, including scanned vehicle titles as a photo or scan. 
- Domain names registered with GoDaddy or other registrar services. It's easy to do and will save you a few dollars maybe. You just have to setup the nameserver (NS) records on the registrar site to point to Amazon. If you search "setup godaddy point to aws" you will find instructions.

## AWS account
![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/AWSfreetier.png)
- If you have a .edu email address you can sign up for an educational account, which will not require a credit card. This will limit your account abilities, since you won't be able to use features that would normally cost.
- To sign up for a free tier, you will have to enter a credit card. Be prepared to be charged a few dollars if you don't pay attention to the terms. If you run more than 750 hours of EC2 instances in a month (there are 744 in a month) you can be charged. If you use more than 30 GB of space, irrelevant of the instance running, you will be charged. 
- Pay attention to [BILLING](https://console.aws.amazon.com/billing/home?region=no-region#/bills) and [cost explorer](https://console.aws.amazon.com/billing/home?region=no-region#/). They will show you what is going to potentially run you charges. Don't panic. Once you remove it, the charges may go away. 

## Identity and Access Management (IAM) & Account Security
- When you have your account created, you're going to come out with a number of pieces of data. Stored these in your password manager! This is for your root account. It is recommended you setup Multi-factor Authentication for your root account.
	- AWS Console address: https://<account-id-number>.signin.aws.amazon.com/console
	- Username & Password
	- Account ID
	- Access Key ID
	- Secret Access Key
- [IAM Dashboard](https://console.aws.amazon.com/iam) will give you security status recomendations.
	- The important recommendation is to create a new Administrator user that will actually be you. You shouldn't be doing everything as root.

## IAM Access to for DNS Record Updates
- 
- lets create a way for us to update our DNS records for the domain name we will have in the future. This is done by creating "system account" and giving "programatic access" to only a few things.
- Create a new IAM Policy [Policies: Create Policy] and in the [JSON] tab copy in the following code:
```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets",
                "route53:GetHostedZone",
                "route53:ListResourceRecordSets"
            ],
            "Resource": [
                "arn:aws:route53:::hostedzone/<hosted-zone-id>"
            ]
        },
        {
            "Action": [
                "route53:ListHostedZones",
                "route53:GetChange"
            ],
            "Effect": "Allow",
            "Resource": [
                "*"
            ]
        }
    ]
}
```


## Create our domain name
 - register domain name via Route53 and create a new hosted zone
 - create a new VPC using [AWS best practices](https://aws.amazon.com/answers/networking/aws-single-vpc-design/)
	- You will likely only need a handfull of IP addresses. 
	- I prefer a Network such as 10.1.1.0/24, Netmask 255.255.255.0, Range 10.1.1.1-10.1.1.254
 - create a new Subnet in the VPC
	- Be sure to assign the correct VPC to the Subnet 
	- The subnet can be from the size of the VPC at /24 up to /28
	- I prefer Subnets with:
		- CIDR block of 10.1.1.0/27, Range 10.1.1.1-10.1.1.30 (call it Public) and 
		- CIDR block of 10.1.1.32/27, Range 10.1.1.33-10.1.1.63 (call if Private)
 - set your Subnet to "Enable auto-assign Public IP"
 - create a Security Group for your VPC
 - create inbound rules such as: HTTP (80), HTTPS (443), SSH (22)

 - create a new Ubuntu EC2 instance
 - download certbot

 - configure nginx.conf files to registered domain
 	- nealalan/EC2_Ubuntu_LEMP/[nginx.servers.conf.txt](./nginx.servers.conf.txt)
 - create a link in your home folder to get to 

 - setup a bash script to automatically run upon instance load to update the DNS record to the correct public IP address
	- nealalan/[update_route53](https://github.com/nealalan/update_route53)
	- This will allow you to ssh into your instance using the domain name instead of the newly assigned public IP address


 - To be continued...
