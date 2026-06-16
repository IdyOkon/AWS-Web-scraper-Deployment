# AWS-Web-scraper-Deployment
High availability web scraping application deployed on AWS EC2 across multiple AZs with monitoring and logging

# Project Overview
This project demonstrates the deployment of a web scraping application on AWS using EC2 instances distributed across multiple Availability Zones (AZs) to ensure high availability and fault tolerance. The system includes automated monitoring, centralized logging, and alerting using CloudWatch and SNS to notify on instance failures, with log archival to Amazon S3 for long-term storage.

# Architecture
- 2 EC2 instances deployed in different Availability Zones (eu-north-1a and eu-north-1b)
- Web scraper application running on both instances
- IAM role attached for secure AWS service access
- CloudWatch for monitoring instance health and logs
- SNS for email alert notifications on failures
- S3 bucket for log archival

  # AWS Services Used
- Amazon EC2 (compute)
- Amazon VPC (network isolation)
- IAM (access control and permissions)
- Amazon CloudWatch (monitoring, logs & alarms)
- Amazon SNS (alert notifications)
- Amazon S3 (log storage and archival)

# Deployment Steps
# 1. Launch EC2 Instances
 - Launch two EC2 instances in different Availability Zones
 - Assign a key pair and security group allowing SSH access

# 2. Install Dependencies
```bash
sudo yum update -y
sudo yum install git -y
sudo yum install python3 -y
sudo yum install python3-pip -y
```

# 3. Clone Repository
```bash
git clone <your-repo-url>
cd Webscraper
```

# 4. Install Python Dependencies
```bash
pip3 install -r requirements.txt
```

# 5. Run Web Scraper
```bash
python3 webscraper.py https://quotes.toscrape.com
```

# 6. Run in Background
```bash
nohup python3 webscraper.py https://quotes.toscrape.com > output.log 2>&1 &
```

# Monitoring & Alerting
# CloudWatch Alarm
- Metric: StatusCheckFailed
- Condition: Greater than 0
- Evaluation: 1 minute interval
- Action: Trigger SNS notification

# SNS Alerting
- SNS topic created: webscraper-alerts
- Email subscription configured and confirmed
- Alerts sent when EC2 instance failure is detected

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
Web scraper deployed across two EC2 instances
Each instance placed in a separate Availability Zone
Ensures system continues running if one AZ fails

# How to Reproduce
- Create VPC with public subnets in two AZs
- Launch 2 EC2 instances in different AZs
- Attach IAM role (webscraper-prod-role)
- Install dependencies and clone repo
- Run scraper on both instances
- Configure CloudWatch alarm + SNS topic
- Confirm email subscription for alerts
- Configure CloudWatch Agent for logging
- Export logs to S3 for archival

# Security Considerations
- IAM roles used instead of hardcoded credentials
- Security groups restrict SSH access to trusted IPs
- Infrastructure designed with least privilege principle

# Outcome
A fully functional, highly available web scraping system deployed on AWS with monitoring, centralized logging, alerting, and long-term log archival.
