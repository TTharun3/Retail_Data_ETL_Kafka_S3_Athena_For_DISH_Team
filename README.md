Step 1: Set Up AWS EC2 Instance

	1.	Log in to AWS:
	•	Create or log in to your AWS account.
	2.	Launch an EC2 Instance:
	•	Navigate to the EC2 Dashboard.
	•	Click Launch Instance.
	•	Choose an Amazon Linux 2 AMI (or Ubuntu).
	•	Select an instance type (e.g., t2.micro for free tier).
	•	Configure the instance details, add storage, and launch the instance with a key pair.
	•	Ensure the security group allows inbound traffic for:
	•	SSH (Port 22)
	•	Kafka (Port 9092)
	•	Zookeeper (Port 2181)
	3.	Connect to Your EC2 Instance:
	•	Use the key pair and SSH into your EC2 instance:

