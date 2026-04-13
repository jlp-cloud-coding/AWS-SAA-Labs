## Demo Steps
1. Create an EC2 instance with t2.micro/t3.micro which is free-tier eligible.
2. Ensure its set to the default VPC and has a public IP and create a security group
3. Optional - Enable detailed monitoring (paid option - you can continue without this, but would take sometime longer)
4. Connect to the instance just created via EC2 Instance Connect from the browser and install stress

5. Create a CloudWatch alarm based on the CPU Utilisation of the created instance
6. For testing the alarm use Threshold greater than or equal to >= 15%
7. Initially alarm status would be "ok":
   
<img width="958" height="356" alt="1" src="https://github.com/user-attachments/assets/473fb608-3fb5-4bca-b4e9-f94e490251fb" />
9. Go back to the EC2 browser console and Install stress using 'sudo yum install stress -y'
10. Run stress 'stress -c 1 -t 3600'
11. Wait for alarm to change from "ok" to "in alarm" (because of running stress we are imposing artificial load on CPU):
    
<img width="959" height="356" alt="in_alarm" src="https://github.com/user-attachments/assets/63167fc9-5173-4666-99fe-8d095f7fcc30" />

12. Use ctrl + c in the ec2 browser console to cancel stress
13. Wait for alarm to return back to "OK"

<img width="959" height="377" alt="in alarm to OK" src="https://github.com/user-attachments/assets/495298f6-3301-4f78-b98a-6ebdd3e62257" />

14. Delete the alarm
15. Delete the EC2 instance
16. Delete the security group created along with ec2 instance
