Step 1: Set Up AWS EC2 Instance

	1.	Log in to AWS:
		•Create or log in to your AWS account.
	2.	Launch an EC2 Instance:
		• Navigate to the EC2 Dashboard.
		• Click Launch Instance.
		• Choose an Amazon Linux 2 AMI (or Ubuntu).
		• Select an instance type (e.g., t2.micro for free tier).
		• Configure the instance details, add storage, and launch the instance with a key pair.
		• Ensure the security group allows inbound traffic for:
		• SSH (Port 22)
		• Kafka (Port 9092)
		• Zookeeper (Port 2181)
	3.	Connect to Your EC2 Instance:
		• Use the key pair and SSH into your EC2 instance: ssh -i "your-key.pem" ec2-user@your-public-ip
  
  Step 2: Install Java

		Kafka requires Java to run. Install Java 8 or higher:

  		sudo yum update -y               # Update packages
		sudo yum install java-1.8.0-openjdk -y   # Install Java
		java -version                    # Verify the installation

  
Step 3: Download and Extract Kafka

	1.	Download the latest version of Kafka: wget https://downloads.apache.org/kafka/3.5.0/kafka_2.13-3.5.0.tgz
 	2.	Extract the tarball: 
  		tar -xvf kafka_2.13-3.5.0.tgz
		cd kafka_2.13-3.5.0
  
Step 4: Start Zookeeper

	Kafka requires Zookeeper to manage its distributed system. Use the default configuration to start Zookeeper:
	1.	Start Zookeeper: bin/zookeeper-server-start.sh config/zookeeper.properties
 	2.	Keep this terminal open.

Step 5: Start Kafka Broker

	1.	Open another SSH session to the EC2 instance.
	2.	Set Kafka’s heap size (optional for resource-limited instances): export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
 	3.	Start the Kafka broker:bin/kafka-server-start.sh config/server.properties

Step 6: Configure Kafka for Public Access

	By default, Kafka listens to private IPs. Update server.properties to use the public IP of your EC2 instance:
	1.	Open the Kafka configuration file: sudo nano config/server.properties
 	2.	Modify the ADVERTISED_LISTENERS property:
  		listeners=PLAINTEXT://:9092
		advertised.listeners=PLAINTEXT://<YOUR_PUBLIC_IP>:9092
  	3.	Save the file and restart the Kafka broker.

   Step 7: Create a Kafka Topic

	1.	Open another SSH session.
	2.	Create a topic: bin/kafka-topics.sh --create --topic demo_topic --bootstrap-server <YOUR_PUBLIC_IP>:9092 --replication-factor 1 --partitions 1

 Step 8: Start a Producer

	1.	Start a Kafka producer: bin/kafka-console-producer.sh --topic demo_topic --bootstrap-server <YOUR_PUBLIC_IP>:9092
 	2.	Type messages into the console. Each line is a new message sent to Kafka.

  Step 9: Start a Consumer

	1.	Open another SSH session.
	2.	Start a Kafka consumer: bin/kafka-console-consumer.sh --topic demo_topic --bootstrap-server <YOUR_PUBLIC_IP>:9092
 	3.	The consumer will display messages sent by the producer in real-time.

  Step 10: Test the Setup

	•	Type messages in the producer console.
	•	Verify they appear in the consumer console.

 Tips for Smooth Operation:

	1.	Security Groups: Ensure your EC2 instance’s security group allows traffic on Kafka (9092) and Zookeeper (2181) ports.
	2.	Firewall: Disable firewalls on the EC2 instance or allow the required ports.
	3.	System Resources: Monitor memory and CPU usage. Upgrade instance type if required




  

