**Retail Data Streaming and Analytics with Apache Kafka, AWS S3, and Athena**

[ETL_Workflow.jpeg](https://github.com/TTharun3/Retail_Data_ETL_Kafka_S3_Athena_For_DISH_Team/blob/main/ETL_Workflow.jpeg)

**Project Overview**

This project demonstrates a complete real-time data pipeline for streaming, storing, and analyzing data. Using the retail dataset, we simulate data streaming via Apache Kafka, store the streamed data into AWS S3, and leverage AWS Glue Crawler and Amazon Athena for analysis. This pipeline is ideal for scenarios requiring scalable, near-real-time data ingestion and analytics

**Workflow**

	1.	Data Streaming with Apache Kafka:
		• A Kafka producer streams data from a retail dataset stored as a CSV file.
		• The data is sent to a Kafka topic in JSON format.
		• The producer simulates real-time streaming by sending one record every second.
	2.	Data Storage in AWS S3:
		• A Kafka consumer listens to the Kafka topic, processes the incoming messages, and writes them to an AWS S3 bucket.
		• Each message is saved as a JSON file in the S3 bucket for further processing.
	3.	Data Crawling and Schema Detection:
		• AWS Glue Crawler scans the S3 bucket, detects the schema of the JSON files, and creates a table in the AWS Glue Data Catalog.
		• This step makes the data queryable in Amazon Athena.
	4.	Data Analysis with Athena:
		• Using SQL queries in Athena, we analyze the retail dataset stored in S3.
		• Examples of analyses:
		• Total sales by region.
		• Top-performing products.
		• Seasonal trends and sales patterns.

**Setup**

**Step 1: Set Up AWS EC2 Instance**

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
  
** Step 2: Install Java**

		Kafka requires Java to run. Install Java 8 or higher:

  		sudo yum update -y               # Update packages
		sudo yum install java-1.8.0-openjdk -y   # Install Java
		java -version                    # Verify the installation

  
**Step 3: Download and Extract Kafka**

	1.	Download the latest version of Kafka: wget https://downloads.apache.org/kafka/3.5.0/kafka_2.13-3.5.0.tgz
 	2.	Extract the tarball: 
  		tar -xvf kafka_2.13-3.5.0.tgz
		cd kafka_2.13-3.5.0
  
**Step 4: Start Zookeeper**

	Kafka requires Zookeeper to manage its distributed system. Use the default configuration to start Zookeeper:
	1.	Start Zookeeper: bin/zookeeper-server-start.sh config/zookeeper.properties
 	2.	Keep this terminal open.

**Step 5: Start Kafka Broker**

	1.	Open another SSH session to the EC2 instance.
	2.	Set Kafka’s heap size (optional for resource-limited instances): export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
 	3.	Start the Kafka broker:bin/kafka-server-start.sh config/server.properties

**Step 6: Configure Kafka for Public Access**

	By default, Kafka listens to private IPs. Update server.properties to use the public IP of your EC2 instance:
	1.	Open the Kafka configuration file: sudo nano config/server.properties
 	2.	Modify the ADVERTISED_LISTENERS property:
  		listeners=PLAINTEXT://:9092
		advertised.listeners=PLAINTEXT://<YOUR_PUBLIC_IP>:9092
  	3.	Save the file and restart the Kafka broker.

**Step 7: Create a Kafka Topic**

	1.	Open another SSH session.
	2.	Create a topic: bin/kafka-topics.sh --create --topic demo_topic --bootstrap-server <YOUR_PUBLIC_IP>:9092 --replication-factor 1 --partitions 1

** Step 8: Start a Producer**

	1.	Start a Kafka producer: bin/kafka-console-producer.sh --topic demo_topic --bootstrap-server <YOUR_PUBLIC_IP>:9092
 	2.	Type messages into the console. Each line is a new message sent to Kafka.

**Step 9: Start a Consumer**

	1.	Open another SSH session.
	2.	Start a Kafka consumer: bin/kafka-console-consumer.sh --topic demo_topic --bootstrap-server <YOUR_PUBLIC_IP>:9092
 	3.	The consumer will display messages sent by the producer in real-time.

**Step 10: Test the Setup**

	•	Type messages in the producer console.
	•	Verify they appear in the consumer console.

**Tips for Smooth Operation:**

	1.	Security Groups: Ensure your EC2 instance’s security group allows traffic on Kafka (9092) and Zookeeper (2181) ports.
	2.	Firewall: Disable firewalls on the EC2 instance or allow the required ports.
	3.	System Resources: Monitor memory and CPU usage. Upgrade instance type if required



**Producer Code Explanation**

	1.	Imports:
		• The script uses pandas to handle data from a CSV file.
		• KafkaProducer from the kafka-python library is used to send messages to a Kafka topic.
		• Other libraries like time and json help manage timing and data serialization.
	2.	Producer Initialization:
		• A Kafka producer is initialized with the bootstrap_servers pointing to the Kafka broker’s public IP and port (9092).
		• value_serializer ensures that data sent to Kafka is serialized into JSON format and encoded in UTF-8.
	3.	Sending Test Data:
		• The producer sends a test message ({'Tharun': 'Ponnaganti'}) to a topic named Trail.
	4.	Streaming Data from a CSV:
		• The script reads a CSV file (Retail.csv) into a Pandas DataFrame.
		• Inside a while loop, the script selects a random row (df.sample(1)), converts it to a dictionary, and sends it to the Kafka topic (demo_test) every second.
		• The producer ensures the messages are streamed continuously by adding a delay using sleep(1).
	5.	Flush Data:
		• At the end of the process, the producer flushes all buffered messages to Kafka.

**Consumer Code Explanation**

	1.	Imports:
		• KafkaConsumer is used to consume messages from the Kafka topic.
		• The json module deserializes messages from JSON format.
		• s3fs enables interaction with AWS S3 buckets.
	2.	Consumer Initialization:
		• A Kafka consumer is initialized to subscribe to the Trail topic.
		• The bootstrap_servers parameter points to the Kafka broker’s public IP and port.
		• value_deserializer converts consumed messages from JSON format into Python objects.
	3.	S3 Bucket Interaction:
		• An S3FileSystem instance is initialized to interact with an S3 bucket.
	4.	Saving Messages to S3:
		• The consumer iterates through messages in the topic (Trail).
		• For each message:
		• It creates a new JSON file in the S3 bucket with the filename format retail_data{count}.json, where count is the message index.
		• The message value is written into the JSON file.


  

