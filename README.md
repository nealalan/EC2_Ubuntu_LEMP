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

![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/iam.png)
- [IAM Dashboard](https://console.aws.amazon.com/iam) will give you security status recomendations.
	- The important recommendation is to create a new Administrator user that will actually be you. You shouldn't be doing everything as root.

## Pick Your Domain Name
 - Domain names registered with GoDaddy or other registrar services. 
 	- Easy to do. 
	- Can save you a few dollars. If you search "setup godaddy point to aws" you will find instructions.
	- You have to setup the nameserver (NS) records on the registrar site to point to Amazon. 
 - For AWS Route 53, Go to "Registered domains" and click "Register domain"

## Create a Hosted Zone in Route 53
- Go to "Hosted Zones" and click "Create Hosted Zone"
	- Enter your domain name: neonaluminum.com
	- Click Create!

![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/nsrecords.png)
- You now have your Start of Authority (SOA) and NameServer (NS) records. The NS records will be entered under the domain as the name servers to look for the DNS records.

## IAM Access for DNS Record Updates
- In the future we will need to update our DNS records for the domain name. 
	- Using a dynamic IP addresses costs extra money, 
	- Using a static IP address and updating a DNS record is free. 
- You will need the "Hosted Zone ID" from Route 53 for this step.

![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/hostedzone.png)
- We want to use IAM to create a "system account" and give "programatic access" to only a few fuctions.
	- Create a new IAM Policy under Policies: Create Policy 
	- Enter a name such as "AmazonRoute53UpdateDNS" and a Description.
	- In the JSON tab, copy in the following code:
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
- Create a New Group called "UpdateRoute53" 
	- Attach Policies "AmazonEC2ReadOnlyAccess" the new one you created "AmazonRoute53UpdateDNS"
- Create a New User called "domain-name_update_dns" and check "Programatic Access"
	- Add user to group "UpdateRoute53"
	- Click Create User
	- IMPORTANT! You will not be given an Access Key ID and Secret Access Key. You can NOT retrieve these later. I recommend you click download .csv file and you copy and paste these into a new entry in your password manager.
![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/accesskey.png)

## Virtual Private Cloud (VPC)
- A VPC is an isolated portion of the AWS cloud populated by AWS objects, such as Amazon EC2 instances. 
- You can read here about using [AWS best practices](https://aws.amazon.com/answers/networking/aws-single-vpc-design/)
- Some things will need to be created in order to launch our instance:
	- VPC : I recommend using the [Start VPC Wizzard](https://us-east-2.console.aws.amazon.com/vpc/home?region=us-east-2#), this will create a lot of what you need!
		- The VPC CIDR Address is somewhat important. Read my next section.
		- Route Table,
		- Internet Gateway,
		- Network ACL,
		- Security Group, 
		- Public Subnet, 
		- Network Interface
- This is a lot to digest, but it's important to understand what is being created and how everything relates. 

## VPC CIDR Address
- The IP addresses used within your cloud is part of a dedicated range of IP addresses. Whenever you connect to a WIFI connection, you'll likely have an IP address that begins with 192.168 or 172. These are ranges in [private network](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces) ranges.

![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/cidrcalc.png)
- Creating a Cloud we will create an [IPv4 CIDR block](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#IPv4_CIDR_blocks) 10.10.10.0/24 will give us address space for a standard small network.

## VPC: Public Subnetwork (Subnet)
- Since the VPC is actually "virtual" we need to create an actual network (even though it's also virtual since it's in the cloud) to place our actual servers (that are also virtual).
- A [subnet](https://en.wikipedia.org/wiki/Subnetwork) can take up the entire address space of the VPC or you can make many subnets within a VPC to allow for private networks and a higher level of security.
	- Since my VPC CIDR block is 10.10.10.0/24, I will create create a smaller subnet in the CIDR block of the VPC, set to 10.10.10.0/27.

## VPC: Security: Network Access Control Lists (ACLs)
- See [VPC ACLs documentation](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html) for full explanation. Here's Amazons brief explanation:
	- A network access control list (ACL) is an optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more subnets. You might set up network ACLs with rules similar to your security groups in order to add an additional layer of security to your VPC. For more information about the differences between security groups and network ACLs, see [Comparison of Security Groups and Network ACLs](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Security.html#VPC_Security_Comparison).
	- There are many different ways to configure security for your instance - Network ACLs, Security Groups, IPTables within the instance, making a subnet private - as you can see in this diagram:

![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/security-diagram.png)

## EC2: Network & Security: Key Pairs
- This Key Pair is not the same as the key that was created when you created the AWS account!
![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/ssh_patrick_Lnx_illustration.png)
- Now, [lets create one](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#KeyPairs:)! (If you have one already, you can upload it also.)
	- This key file that will be used to connect to your EC2 instances via SSH.
	- Name it something more specific than "key".
	- Creating the key now, will have you automatically download it.
	- You can assign this key to an instance later. 
	- Note: If you wait and let the key creation happen at the time you create an EC2 instance, you may run into frustrations. 
	- Note Conclusion: It's just easier to create the key pair now and download your .pem file.
- Store your .pem file in your home/ folder or ~/.ssh/ folder

## LAUNCH INSTANCE!
- Please don't jump through this. Go through each screen by clicking "Next" and not "Lanuch"
- From the [EC2 Dashboard](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2), "Launch Instance" 
![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/ubuntuami.png)
- Step 1: Choose an Amazon Machine Image (AMI)
	- You'll want to scroll down to "Ubuntu Server" and make sure it says "Free tier eligible"
	- Select
- Step 2: Choose an Instance Type
	- "Free tier eligible"
	- "Next: Configure Instance Details"
- Step 3: Configure Instance Details
	- Purchasing Option: You're in the free tier, no need to check this. 
	- Network: Select the VPC you created
	- Subnet: Select the Subnet you created
	- Auto-assign Public IP: This should be Enabled by Subnet default
	- Likely don't change anything else unless you want charges.
	- Network interfaces: Allow a new network interface to be created for the subnet
	- Next: Add Storage
- Step 4: Add Storage
	- Next: Add Tags
- Step 5: Add Tags
	- Add Tag, Key = "Name", Value = "Neals Web Server"
	- Next: Configure Security Group
- Step 6: Configure Security Group
	- Security Group Name: "Neals Public Subnet SG"
	- Since we created rules in the ACL earlier, those should limit traffic.
	- One Rule: Type = All Traffic, Source = Custom 0.0.0.0/0
	- Review and Launch
- Step 7: Review Instance Launch
	- Launch
	- Select "Choose an existing key pair" and select the key pair you created earlier.
- It'll take a few minutes for the server to get up and running. You now have an Ubuntu server running in a Virtual Private Cloud, within a Public Subnet controlled by an ACL that only lets in SSH and HTTPS traffic.

![](https://raw.githubusercontent.com/nealalan/EC2_Ubuntu_LEMP/master/ec2instanceline.png)
- Go back to [EC2 Instance](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Instances:sort=desc:publicIp) dashboard and select your running instance, you can see your IPv4 Public IP address. This is what you can use to connect to your server.
- Click on "Actions: Instant State" for the options to Start and Stop your instances. 

## CONNECT TO YOUR INSTANCE
- Command line: ssh -i ~/.ssh/neals_web_server.pem ubuntu@<ip-address>
- Amazon also has a way to connect via the web browser through a Java plugin.
```bash
# The first thing you want to do is ensure you're upgraded fully
ubuntu@ip-10-10-10-13:~$ sudo apt -y update; sudo apt -y upgrade
```

## Certbot
- download certbot

## NGINX Configuration
 - configure nginx.conf files to registered domain
 	- nealalan/EC2_Ubuntu_LEMP/[nginx.servers.conf.txt](./nginx.servers.conf.txt)
 - create a link in your home folder to get to 

## Auto Update Route 53
 - setup a bash script to automatically run upon instance load to update the DNS record to the correct public IP address
	- nealalan/[update_route53](https://github.com/nealalan/update_route53)
	- This will allow you to ssh into your instance using the domain name instead of the newly assigned public IP address


 - To be continued...

[edit](https://github.com/nealalan/EC2_Ubuntu_LEMP/edit/master/README.md)
