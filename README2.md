# Secure Remote Access to AWS EC2 Instances

There is a lot to consider when establishing processes to securely connect to your EC2 instances. IAM roles and permissions, Security Groups to allow SSH access, bastion hosts to connect through, source IPs to restrict incoming traffic from, and SSH keys to create, rotate, and expire.

This demonstration will show one example of how to use AWS Systems Manager Session Manager and EC2 Instance Connect together to access your EC2 instances in a manner that addresses most of the concerns above.

Reference Documents
===
* [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
* [EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Connect-using-EC2-Instance-Connect.html)


AWS Resource Requirements
===
* EC2 instances should have the latest version of ssm-agent installed
* EC2 instances should have the latest version of ec2-instance-connect installed
* EC2 instances need to be able to communicate outbound to the public AWS Systems Manager Endpoint
* EC2 instances need to have an appropriate policy attached that allows access to AWS Systems Manager
* The user connecting to the EC2 instance needs to have an appropriate policy attached that allows access to AWS Systems Manager and which allows the user to push a temporary public SSH key to the instance

CLI Requirements
===

You can try using [my AWS IDE](https://github.com/mjuettner/aws-ide) to set up your client environment instead of building your own. The environment created there is literally what was used to build this demo.  If you'd rather set up your own:

* You should have a working [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) environment
* You should have the [Session Manager Plugin for the AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html) installed
* You should have the [EC2 Instance Connect CLI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-set-up.html#ec2-instance-connect-install-eic-CLI) installed

