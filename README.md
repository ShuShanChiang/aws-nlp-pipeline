# AWS NLP Pipeline – WordFreq Application

This project showcases a scalable, event-driven NLP pipeline on AWS for processing text uploads and extracting the top 10 most frequent words per file. The solution leverages AWS services such as S3, Lambda, EC2, SQS, DynamoDB, CloudWatch, and Auto Scaling to demonstrate cost-effective and resilient architecture for real-time analytics.

---

## Architecture Overview

- **User Upload**: Text files are uploaded from an on-premise source to an S3 Uploading Bucket.
- **Event Trigger**: Each upload triggers an S3 Event Notification to an SQS queue.
- **Processing**: EC2 instances (configured with Auto Scaling) read from the queue, process files, and compute word frequencies.
- **Result Storage**: Output is stored in DynamoDB and sent to a second SQS queue.
- **Monitoring**: CloudWatch tracks queue depth and triggers Auto Scaling adjustments.
- **Backup & Resilience**: AWS Backup, S3 Glacier, and Multi-AZ configurations ensure durability.
![image](https://github.com/user-attachments/assets/5c75cda5-0d6d-46b3-bcb9-617a7721bbb4)

---

## Technologies Used

- **Amazon S3** – File storage and versioning
  ![image](https://github.com/user-attachments/assets/b7735fc9-8e0d-45fe-8b2c-ef4d3921866e)
  
- **Amazon SQS** – Decoupled messaging for task queues
  ![image](https://github.com/user-attachments/assets/2eb2b481-8693-43da-ab63-4a90a9d0b313)
  ![image](https://github.com/user-attachments/assets/8e0493e3-1de6-427d-b028-320a49d2917e)

- **Amazon EC2** – Compute for processing jobs
  ![image](https://github.com/user-attachments/assets/c689972e-5700-4f04-b925-2c721950c281)
  ![image](https://github.com/user-attachments/assets/09bf0ab5-0162-4150-9e41-517ba2f9cb10)

- **Auto Scaling Group** – Elastic scaling based on queue depth
  ![image](https://github.com/user-attachments/assets/6ce70be9-d1f3-43e0-8662-a33566e2e325)
  ![image](https://github.com/user-attachments/assets/0b81c639-4780-470e-8b23-544c0b37f42f)

- **Amazon CloudWatch** – Metrics, alarms, and logging  
- **AWS Lambda** – Backend automation and notification  
- **Amazon DynamoDB** – Fast NoSQL storage for job results(store the output of top 10 most common words)
  ![image](https://github.com/user-attachments/assets/34e71532-6fea-4b08-ac3f-232e3e493fde)
  
- **AWS Backup / Glacier** – Long-term backup and disaster recovery  
- **Amazon Route 53 / CloudFront / IAM** – Website delivery, security, and access control  

---

## Load Testing & Optimisation

### Auto Scaling Strategy
- CloudWatch alarms trigger scale-up if queue > 6 messages, and scale-down if < 2.
- Test scenarios included:
  - **Different instance types** (e.g., `t2.micro`, `t3.micro`, `t2.large`)
  - **Scaling policies with delay adjustments**
  - **Initial capacity tuning (e.g., starting with 2 instances)**

### Results Summary
- **Baseline (t2.micro)**: 12 min for 120 files  
- **Upgraded Memory (t2.small)**: 9 min (↓ 25%)  
- **Upgraded CPU (t3.micro)**: 5 min (↓ 58%)  
- **Upgraded CPU + RAM (t2.large)**: 4 min (↓ 66%)  
- Demonstrated that CPU was the primary bottleneck, and combining upgrades yielded optimal results.

---

## Further Enhancements

- **Serverless architecture**: Replace EC2 with Lambda for lightweight workloads  
- **Cost optimisation**: Use Spot Instances, Lifecycle policies for S3 → Glacier  
- **Security**: Enforce IAM roles, VPC isolation, KMS encryption  
- **Scalability**: Evaluate Amazon EMR (Hadoop/Spark) for large-scale workloads

---

## Repository Structure

```plaintext
aws-nlp-pipeline/
├── cmd/                            # Entry points for running different services
│   ├── main.go                     # Starts the full pipeline
│   ├── main_Upload.go             # Handles file uploads
│   └── main_createTable.go        # Initializes DynamoDB tables
├── pkg/
│   ├── worker/                    # Message parsing and queueing logic
│   │   ├── config.go
│   │   ├── job_message_parse.go
│   │   ├── job_message_queue.go
│   │   └── worker.go
│   └── result/                    # Result processing and notification
│       ├── result_recorder.go
│       ├── result_collector.go
│       └── result_notifier.go
└── README.md


