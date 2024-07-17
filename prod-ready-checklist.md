### Production-Ready AWS Architecture Checklist

#### **1. Assessment and Planning**

- **Stakeholders Identified**
  - [ ] Management
  - [ ] Developers
  - [ ] Security Team

- **Business Requirements**
  - [ ] Scalability Requirements
  - [ ] Availability Requirements
  - [ ] Performance Requirements

- **Security and Compliance Needs**
  - [ ] Compliance Standards (e.g., GDPR, HIPAA)
  - [ ] Security Policies and Procedures

- **Architecture Design**
  - [ ] High-Level Architecture Diagram Created
    - **Example Tool:** Lucidchart, Draw.io
  - [ ] AWS Services Identified
    - **Example Services:** EC2, RDS, S3, VPC, IAM

#### **2. Infrastructure as Code (IaC) Setup**

- **Terraform Setup**
  - [ ] Terraform Installed
    - **Command:** `terraform -version`
  - [ ] Terraform Project Structure Created
    - **Example Structure:**

      ```plaintext
      ├── terraform
      │   ├── main.tf
      │   ├── variables.tf
      │   ├── outputs.tf
      │   ├── vpc.tf
      │   ├── ec2.tf
      │   └── s3.tf
      ```

  - [ ] Providers Defined
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      provider "aws" {
        region = "us-west-2"
      }
      ```

      </details>

- **Version Control**
  - [ ] GitHub Repository Created
    - **Command:** `gh repo create my-repo --public --confirm`
  - [ ] .gitignore Configured
    - <details>
      <summary>Example .gitignore</summary>

      ```plaintext
      .terraform/
      terraform.tfstate
      terraform.tfstate.backup
      ```

      </details>
  - [ ] Terraform Configurations Stored
    - **Commands:**

      ```bash
      git init
      git add .
      git commit -m "Initial commit with Terraform configurations"
      git remote add origin <repo_url>
      git push -u origin master
      ```

#### **3. Networking and Security**

- **VPC Configuration**
  - [ ] VPC Created
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_vpc" "main" {
        cidr_block = "10.0.0.0/16"
        tags = {
          Name = "main-vpc"
        }
      }
      ```

      </details>
  - [ ] Subnets Created
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_subnet" "subnet1" {
        vpc_id     = aws_vpc.main.id
        cidr_block = "10.0.1.0/24"
        availability_zone = "us-west-2a"
        tags = {
          Name = "subnet-1"
        }
      }
      ```

      </details>
  - [ ] Route Tables Configured
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_route_table" "main" {
        vpc_id = aws_vpc.main.id
        route {
          cidr_block = "0.0.0.0/0"
          gateway_id = aws_internet_gateway.main.id
        }
      }
      ```

      </details>
  - [ ] Security Groups Configured
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_security_group" "web_sg" {
        name_prefix = "web-"
        description = "Allow web traffic"
        vpc_id      = aws_vpc.main.id
        
        ingress {
          from_port   = 80
          to_port     = 80
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
        }
        
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      }
      ```

      </details>

- **IAM Setup**
  - [ ] IAM Roles Defined
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_iam_role" "example" {
        name = "example-role"
        assume_role_policy = jsonencode({
          Version = "2012-10-17"
          Statement = [
            {
              Effect = "Allow"
              Principal = {
                Service = "ec2.amazonaws.com"
              }
              Action = "sts:AssumeRole"
            },
          ]
        })
      }
      ```

      </details>
  - [ ] IAM Policies Defined
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_iam_policy" "example" {
        name        = "example-policy"
        description = "A test policy"
        policy      = jsonencode({
          Version = "2012-10-17"
          Statement = [
            {
              Action = [
                "ec2:Describe*",
              ]
              Effect   = "Allow"
              Resource = "*"
            },
          ]
        })
      }
      ```

      </details>
  - [ ] Least Privilege Access Configured
    - **Best Practice:** Regularly review IAM roles and policies to ensure they follow the principle of least privilege

#### **4. Compute and Storage**

- **EC2 Instances**
  - [ ] EC2 Configurations Defined in Terraform
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_instance" "web" {
        ami           = "ami-0c55b159cbfafe1f0"
        instance_type = "t2.micro"
        
        tags = {
          Name = "WebServer"
        }
      }
      ```

      </details>
  - [ ] Secure AMIs Used
    - **Best Practice:** Use only official and updated AMIs
  - [ ] SSH Key Pairs Configured
    - **Command:** `aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem`

- **Storage Solutions**
  - [ ] S3 Buckets Set Up
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_s3_bucket" "bucket" {
        bucket = "my-tf-test-bucket"
        acl    = "private"
      }
      ```

      </details>
  - [ ] EBS Volumes Configured
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_ebs_volume" "example" {
        availability_zone = "us-west-2a"
        size              = 40
      }
      ```

      </details>
  - [ ] Proper Permissions Set for S3 Buckets
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_s3_bucket_policy" "bucket_policy" {
        bucket = aws_s3_bucket.bucket.id
        policy = jsonencode({
          Version = "2012-10-17"
          Statement = [
            {
              Action    = "s3:GetObject"
              Effect    = "Allow"
              Principal = "*"
              Resource  = "${aws_s3_bucket.bucket.arn}/*"
            },
          ]
        })
      }
      ```

      </details>

#### **5. Continuous Integration and Deployment (CI/CD)**

- **GitHub Actions Setup**
  - [ ] CI/CD Workflows Created
    - <details>
      <summary>Example Configuration</summary>

      ```yaml
      name: CI/CD Pipeline

      on: [push]

      jobs:
        build:
          runs-on: ubuntu-latest
          steps:
            - uses: actions/checkout@v2
            - name: Set up JDK 11
              uses: actions/setup-java@v2
              with:
                java-version: '11'
            - name: Build with Gradle
              run: ./gradlew build
      ```

      </details>
  - [ ] Build Steps Defined
  - [ ] Test Steps Defined
    - **Example:** Run unit tests using GitHub Actions
  - [ ] Deploy Steps Defined
    - **Example:** Deploy to AWS using GitHub Actions

- **Docker Integration**
  - [ ] Docker Images Built
    - <details>
      <summary>Example Dockerfile</summary>

      ```dockerfile
      FROM openjdk:11-jdk-slim
      COPY . /app
      WORKDIR /app
      RUN ./gradlew build
      CMD ["java", "-jar", "build/libs/myapp.jar"]
      ```

      </details>
  - [ ] Docker Images Stored in AWS ECR
    - **Commands:**

      ```bash
      aws ecr create-repository --repository-name my-repo
      aws ecr get-login-password | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
      docker build -t my-repo .
      docker tag my-repo:latest <account-id>.dkr.ecr.<region>.amazonaws.com/my-repo:latest
      docker push <account-id>.dkr.ecr.<region>.amazon

aws.com/my-repo:latest
      ```

#### **6. Monitoring and Logging**

- **CloudWatch Setup**
  - [ ] CloudWatch Monitoring Configured
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_cloudwatch_log_group" "example" {
        name              = "example-log-group"
        retention_in_days = 14
      }
      ```

      </details>
  - [ ] CloudWatch Logs Set Up
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_cloudwatch_log_stream" "example" {
        name           = "example-log-stream"
        log_group_name = aws_cloudwatch_log_group.example.name
      }
      ```

      </details>
  - [ ] CloudWatch Alarms Configured
    - <details>
      <summary>Example Configuration</summary>

      ```hcl
      resource "aws_cloudwatch_metric_alarm" "cpu_alarm" {
        alarm_name          = "cpu_alarm"
        comparison_operator = "GreaterThanOrEqualToThreshold"
        evaluation_periods  = "1"
        metric_name         = "CPUUtilization"
        namespace           = "AWS/EC2"
        period              = "300"
        statistic           = "Average"
        threshold           = "80"
        alarm_actions       = [aws_sns_topic.topic.arn]
      }
      ```

      </details>

#### **7. Security Best Practices**

- **Encryption**
  - [ ] Data at Rest Encrypted
  - [ ] Data in Transit Encrypted

- **Security Audits**
  - [ ] Regular Security Reviews Scheduled
  - [ ] Security Automation Implemented (e.g., AWS Config)

#### **8. Testing**

- **Unit Testing**
  - [ ] Use a framework like JUnit, pytest, or similar

- **Integration Testing**
  - [ ] Use tools like Postman or Newman

- **End-to-End Testing**
  - [ ] Use Selenium, Cypress, or similar

- **Load Testing**
  - [ ] Use JMeter or Gatling

- **Security Testing**
  - [ ] Use tools like OWASP ZAP or Nessus

#### **9. Documentation and Handover**

- **Documentation**
  - [ ] Infrastructure Setup Documented
  - [ ] Runbooks for Common Tasks Created

- **Handover**
  - [ ] Training Sessions Conducted
  - [ ] Knowledge Transfer Completed

### Checklist Usage

- [ ] Review each section to ensure all steps are completed
- [ ] Validate configurations with peer reviews
- [ ] Test thoroughly before production deployment
- [ ] Regularly update the checklist based on new requirements or tools
