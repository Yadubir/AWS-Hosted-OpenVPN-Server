
# VPN Server Setup Documentation

## Project Title: AWS-Hosted OpenVPN Server with Automated Deployment

### Overview
So recently I was exploring and I got to know about OpenVPN and how we can make our own VPN server using it. I came across some challenges but was able to host it on an EC2 instance and automated it using Terraform. So, this document provides a step-by-step guide for setting up a secure VPN server on AWS EC2 using OpenVPN. It includes automation of the deployment process with Terraform, implementation of security measures, and monitoring of server activity.

### Prerequisites
- An AWS account with appropriate permissions
- AWS CLI installed and configured
- Terraform installed on your local machine

### Setup Instructions

1. **Provisioning the EC2 Instance**
   - Create a Terraform configuration file named `main.tf` with the following content:
   ```hcl
   provider "aws" {
     region = "us-west-2"  # Change to your region
   }

   resource "aws_instance" "openvpn_server" {
     ami           = "<YOUR_AMI_ID>"  # Use a compatible AMI ID
     instance_type = "t2.micro"       # Select instance type (e.g., t2.micro for low-cost)
     tags = {
       Name = "OpenVPN-Server"
     }
     security_groups = ["OpenVPN-SecurityGroup"]
   }

   resource "aws_security_group" "OpenVPN_SG" {
     name        = "OpenVPN-SecurityGroup"
     description = "Allow OpenVPN and SSH"

     ingress {
       from_port   = 1194
       to_port     = 1194
       protocol    = "udp"
       cidr_blocks = ["0.0.0.0/0"]
     }

     ingress {
       from_port   = 22
       to_port     = 22
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"]
     }
   }
   ```

   - Initialize and apply the Terraform configuration:
   ```bash
   terraform init
   terraform apply
   ```

2. **Monitoring and Logging with CloudWatch**
   - Install the CloudWatch agent on your EC2 instance:
   ```bash
   sudo yum install amazon-cloudwatch-agent -y
   ```
   - Create a CloudWatch agent configuration file at `/opt/aws/amazon-cloudwatch-agent/bin/config.json`:
   ```json
   {
     "logs": {
       "logs_collected": {
         "files": {
           "collect_list": [
             {
               "file_path": "/var/log/openvpn.log",
               "log_group_name": "OpenVPNLogs",
               "log_stream_name": "{instance_id}"
             }
           ]
         }
       }
     }
   }
   ```
   - Start the CloudWatch agent:
   ```bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a start -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
   ```

3. **Enhancing Security**
   - Set up fail2ban to prevent unauthorized access:
   ```bash
   sudo yum install fail2ban -y
   ```
   - Configure fail2ban for OpenVPN in `/etc/fail2ban/jail.local`:
   ```ini
   [openvpn]
   enabled  = true
   port     = 1194
   filter   = openvpn
   logpath  = /var/log/openvpn.log
   maxretry = 3
   ```
   - Restart fail2ban:
   ```bash
   sudo systemctl restart fail2ban
   ```

### Conclusion
This guide provides a comprehensive approach to deploying a secure VPN server on AWS. The integration of Terraform for automation and CloudWatch for monitoring ensures a robust setup ready for secure remote access.
