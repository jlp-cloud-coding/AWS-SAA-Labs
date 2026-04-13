## Demo Steps
Create an EC2 instance with t2.micro/t3.micro which is free-tier eligible
Ensure its set to the default VPC and has a public IP and create a security group
Optional - Enable detailed monitoring (paid option - you can continue without this, but would take sometime longer)
Connect to the instance just created via EC2 Instance Connect from the browser and install stress
Install stress using 'sudo yum install stress -y'
Create a CloudWatch alarm based on the CPU Utilisation of the created instance
For testing use Threshold greater than or equal to >= 15%
Run stress 'stress -c 1 -t 3600'
Initially alarm status would be "ok"
<img width="958" height="356" alt="1" src="https://github.com/user-attachments/assets/473fb608-3fb5-4bca-b4e9-f94e490251fb" />
Wait for alarm to change from "ok" to "in alarm" (because of running stress we are imposing artificial load on CPU)

Use ctrl + c in the ec2 browser console to cancel stress
Wait for alarm to return back to "OK"
Delete the alarm
Delete the instance
Delete the security group created while ec2 instance creation
