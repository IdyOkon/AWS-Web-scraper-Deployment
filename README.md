# AWS-Web-scraper-Deployment
High availability web scraping application deployed on AWS EC2 across multiple AZs with monitoring and logging

# Project Overview
This project demonstrates the deployment of a web scraping application on AWS using EC2 instances distributed across multiple Availability Zones (AZs) to ensure high availability and fault tolerance. The system includes automated monitoring, centralized logging, and alerting using CloudWatch and SNS to notify on instance failures, with log archival to Amazon S3 for long-term storage.

# Architecture
The application was designed with availability and observability in mind. The following AWS resources were used to support deployment, monitoring, and log retention:
- 2 EC2 instances deployed in different Availability Zones (eu-north-1a and eu-north-1b): This ensures the     application remains available even if one Availability Zone becomes unavailable.
- Web scraper application running on both instances
- IAM role attached for secure AWS service access
- CloudWatch for monitoring instance health and logs: Used to track instance health, collect application       logs, and create alarms.
- SNS for email alert notifications on failures: Delivers alerts when CloudWatch detects infrastructure        issues.
- S3 bucket for log archival: Stores exported CloudWatch logs for long-term retention and future analysis.

# AWS Services Used
  This project leverages several AWS services to provide a reliable, observable, and fault-tolerant            environment.
  - Amazon EC2 (compute): Hosts the web scraper application.
  - Amazon VPC (network isolation): Provides network isolation and secure communication between resources.
  - IAM (access control and permissions): Used to securely grant permissions to AWS resources.
  - Amazon CloudWatch (monitoring, logs & alarms): Provides monitoring, alarms, and centralized log              collection.
  - Amazon SNS (alert notifications): Sends alert notifications when failures occur.
  - Amazon S3 (log storage and archival): Stores exported logs for long-term retention.

# Deployment Steps
**1. Launch EC2 Instances**
 - Create and launch two EC2 instances in different Availability Zones
 - Assign a key pair and security group allowing SSH access

**2. Connect to EC2 and Install Dependencies**

 From your local machine, SSH into the EC2 instance:

```bash
ssh -i "your-key.pem" ec2-user@<EC2-PUBLIC-IP>

Then install the required dependencies:

```bash
sudo yum update -y
sudo yum install git -y
sudo yum install python3 -y
sudo yum install python3-pip -y
```

**3. Clone Repository**
```bash
git clone <your-repo-url>
cd Webscraper
```

**4. Install Python Dependencies**
```bash
pip3 install -r requirements.txt
```

**5. Run Web Scraper**
```bash
python3 webscraper.py https://quotes.toscrape.com
```

**6. Run in Background**
```bash
nohup python3 webscraper.py https://quotes.toscrape.com > output.log 2>&1 &
```

# Monitoring & Alerting

To ensure operational visibility and rapid incident response, monitoring and alerting were configured using Amazon CloudWatch and Amazon SNS.

 **CloudWatch Alarm**

- Metric: "StatusCheckFailed" - 
It monitors the health status of the EC2 instances and detects infrastructure-level failures.


- Condition: Greater than 0 -
  The alarm is triggered whenever an instance fails a status check.

- Evaluation Period: 1 minute - Checks the instance every minute so problems can be detected quickly.


- Action: Trigger SNS notification - 
Sends an email notification when a failure is detected.

**SNS Alerting**

- SNS Topic: The topic here is "webscraper-alerts"- It sends an email notification when CloudWatch Alarm is triggered.
- Email Subscription: Configured and Confirmed - Receives alerts whenever an EC2 instance fails a status check.

- Automatic Failure Notifications: Alerts are sent automatically when a CloudWatch alarm is triggered, without needing manual checks.


# Logging & Log Archival (CloudWatch -- S3)
Application logs from the web scraper are generated on each EC2 instance and written to a local file **(output.log)** using **nohup** to ensure continuous background execution. Logs were validated in real time using **tail -f output.log**

The Amazon CloudWatch Agent was installed and configured to collect logs from:
```bash
/home/ec2-user/WebScraper/webscraper/output.log
```

These logs are streamed into a centralized CloudWatch Log Group:
```bash
webscraper-logs
```

For long-term retention and archival, CloudWatch Logs were exported to an Amazon S3 bucket:
```bash
web-demo-bucket-20265
```
in which a  CloudWatch export tasks with a defined time range was used (e.g., last 1 hour) and an optional prefix **(cloudwatch-exports).**

This establishes a full observability pipeline:
```bash
EC2 Application → CloudWatch Logs → S3 Archive
```

# High Availability Setup
- Web scraper deployed across two EC2 instances
- Each instance placed in a separate Availability Zone
- Ensures system continues running if one AZ fails

# How to Reproduce
- Create VPC with public subnets in two AZs: To reproduce this deployment, first create a VPC with public      subnets spanning at least two Availability Zones.
- Launch the EC2 instances: Create two EC2 instances and place each one in a separate Availability Zone.
- Configure IAM permissions: Create and attach the required IAM role with CloudWatch, SNS, S3, and Systems     Manager permissions.
- Connect to the EC2 instance using SSH: From your local machine, connect to each instance using SSH to begin setup work.
- Install dependencies: Install Python, Git, and any packages required by the scraper. 
- Clone and run the application: Clone the repository and verify that the scraper can successfully crawl the target website.
- Enable background execution: Run the scraper using **nohup** and redirect output to **output.log** for       continuous operation.
- Configure CloudWatch alarm + SNS topic.
- Confirm email subscription for alerts.
- Configure CloudWatch Agent for logging: Install and configure the Amazon CloudWatch Agent to collect logs    from the application log file and send them to a CloudWatch Log Group.
- Archive logs to S3: Export CloudWatch Logs to an S3 bucket to provide long-term log retention.
- Verify the deployment: Confirm that the scraper runs successfully, logs appear in CloudWatch, alerts are     delivered, and log exports are stored in S3.

# Security Considerations

The deployment was designed with basic cloud security best practices in mind.

- IAM Roles Attached to EC2 instances: AWS permissions were provided through an IAM role instead of storing access keys on the EC2 instances.

- Security Group Restrictions: The EC2 security group was configured to allow SSH access only from authorized IP addresses.

- CloudWatch Agent Permissions: The CloudWatch Agent used the attached IAM role to send logs to CloudWatch.

- S3 and SNS Access Through IAM Policies: Permissions required for log export and notifications were granted through IAM policies attached to the IAM role.

# Outcome

The project successfully demonstrates several core AWS and DevOps concepts:

- High Availability Deployment: The scraper runs across multiple Availability Zones to improve fault tolerance.

- Centralized Monitoring: CloudWatch provides visibility into instance health and application activity.

- Automated Alerting: SNS delivers real-time notifications when failures are detected.

- Centralized Logging: Application logs are collected and managed through CloudWatch Logs.

- Long-Term Log Retention: Logs are archived to Amazon S3 for future analysis and compliance purposes.

- Production-Style Observability Pipeline: The solution combines monitoring, alerting, logging, and storage into a single workflow.