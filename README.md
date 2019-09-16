# AWS SSM with EC2 Instance Connect Demo

This is a CloudFormation template to demonstrate how to connect to AWS EC2 Instances using Systems Manager Session Manager along with EC2 Instance Connect.

There are too many situations to account for to provision perfect IAM roles and policies suited to your environment, so we'll take some shortcuts around those to focus on the SSM and EC2 Instance Connect pieces.

## Prerequisites

In your client environment, install:
- AWS CLI
- Session Manager Plugin for the AWS CLI
- EC2 Instance Connect CLI

In AWS
- Have access to a highly privileged user to run CLI commands

You can try using [my AWS IDE](https://github.com/mjuettner/aws-ide) to maybe simplify setting up your client environment,  what you see there is literally what I used for this demo.

## Setup

Deploy the ssm-ec2ic-demo.yaml CloudFormation template, it will create:
  - Some basic infrastructure:  VPC, Public Subnet, Private Subnet, Internet Gateway, and NAT Gateway
  - Some IAM Resources: A Policy and Role to provide the EC2 instance access to SSM
  - An EC2 Instance: This will reside on the private subnet and it will install the latest SSM Agent

After deploying the CloudFormation template, you should be able to connect to the EC2 instance using SSM:
```
vagrant@ubuntu-bionic:~$ aws ssm start-session --target i-083e4fe8db85c8777

Starting session with SessionId: <username>@<domain>-058d1b1eb3349f5c8
sh-4.2$ 
```

Now, to get EC2 Instance Connect Working with SSM as well, configure your local ssh client to run a command to establish a ProxyTunnel when ssh'ing to EC2 Instances.  In my Ubuntu development environment, that involved modifying my users .ssh/config like so:
```
vagrant@ubuntu-bionic:~$ cat .ssh/config
# SSH over Session Manager
host i-* mi-*
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```

To get SSM and EC2 Instance Connect working together smoothly, we need to make a small change to the EC2 Instance Connect CLI.  The normal usage for EC2 Instance Connect is to execute `mssh <instance id>` which will generate a temporary SSH key, push that key to the EC2 instance metadata, and then execute `ssh <instance ip address>` using the **IP address** of the instance.  That's no good for us, because our EC2 instance is on a private subnet, it won't work.

However, if we modify the `mssh` command to ultimately execute `ssh <instance id>`, that command will trigger the SSH customization we just did above, which will run that ProxyCommand to establish an SSH tunnel to our private instance using SSM, then we'll SSH over that tunnel.

You can [read my issue regarding this](https://github.com/aws/aws-ec2-instance-connect-cli/issues/5) and then do what it says.  Find EC2InstanceConnectCommand.py on your system and change that **host_info** line to **instance_id**.  It's near the very end of the file.

Now that that's done, and you already verified above that just using SSM works, we can bring it all together.

Using SSH:
```
vagrant@ubuntu-bionic:~$ mssh i-083e4fe8db85c8777
Last login: Mon Sep 16 21:08:14 2019 from localhost

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
[ec2-user@ip-10-12-52-231 ~]$
```

Using SFTP:
```
vagrant@ubuntu-bionic:~$ msftp i-083e4fe8db85c8777
Connected to i-083e4fe8db85c8777.
sftp> 
```