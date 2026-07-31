# Tamil Cloudbee AWS Lambda Demo


# AWS Lambda Automation for Scheduled EC2 Start/Stop

This project demonstrates how to automatically **start** and **stop** EC2 instances based on tags using **AWS Lambda**, **IAM**, and **Amazon EventBridge Scheduler**.

## Architecture

```
Amazon EventBridge Scheduler
            │
            ▼
      AWS Lambda Function
            │
            ▼
Describe EC2 Instances by Tags
            │
            ▼
 Start / Stop Matching Instances
```

---

# Prerequisites

- AWS Account
- EC2 instances
- Appropriate IAM permissions to create IAM roles, Lambda functions, and EventBridge schedules

---

# Step 1: Tag Your EC2 Instances

Tag every EC2 instance that should be automatically started and stopped.

| Key | Value |
|------|-------|
| AutoSchedule | True |
| Environment | Dev |

Example:

```
AutoSchedule = True
Environment = Dev
```

Only instances with these tags will be managed.

---

# Step 2: Create IAM Policy for Start Lambda

Navigate to:

```
IAM → Policies → Create Policy
```

Use the following policy:

```json
{
    "Version": "2012-10-17",
    "Statement":[
        {
            "Effect":"Allow",
            "Action":[
                "ec2:DescribeInstances",
                "ec2:StartInstances"
            ],
            "Resource":"*"
        }
    ]
}
```

Save as:

```
Lambda-Start-EC2-Policy
```

---

# Step 3: Create IAM Policy for Stop Lambda

Create another policy.

```json
{
    "Version":"2012-10-17",
    "Statement":[
        {
            "Effect":"Allow",
            "Action":[
                "ec2:DescribeInstances",
                "ec2:StopInstances"
            ],
            "Resource":"*"
        }
    ]
}
```

Save as:

```
Lambda-Stop-EC2-Policy
```

---

# Step 4: Create IAM Role for Start Lambda

Navigate to

```
IAM → Roles → Create Role
```

Trusted Entity

```
AWS Service
```

Use Case

```
Lambda
```

Attach the following policies:

- AWSLambdaBasicExecutionRole
- Lambda-Start-EC2-Policy

Role Name

```
Lambda-Start-EC2-Role
```

---

# Step 5: Create IAM Role for Stop Lambda

Repeat the above steps.

Attach

- AWSLambdaBasicExecutionRole
- Lambda-Stop-EC2-Policy

Role Name

```
Lambda-Stop-EC2-Role
```

---

# Step 6: Create Start Lambda Function

Navigate to

```
Lambda → Create Function
```

Configuration

```
Function Name

Start-EC2-Instances

Runtime

Python 3.x

Execution Role

Use Existing Role

Lambda-Start-EC2-Role
```

Paste the Python code for starting EC2 instances.

Deploy the function.

---

# Step 7: Create Stop Lambda Function

Create another Lambda.

```
Function Name

Stop-EC2-Instances
```

Execution Role

```
Lambda-Stop-EC2-Role
```

Paste the Python code for stopping EC2 instances.

Deploy the function.

---

# Step 8: Test the Lambda Functions

Click

```
Test
```

Verify

- Start Lambda starts stopped instances.
- Stop Lambda stops running instances.

Check CloudWatch Logs for execution details.

---

# Step 9: Create EventBridge Scheduler for Start

Navigate to

```
Amazon EventBridge
→ Scheduler
→ Create Schedule
```

Schedule Name

```
Start-Dev-Servers
```

Schedule Type

```
Recurring
```

Cron Expression

```
cron(0 8 ? * MON-FRI *)
```

Target

```
AWS Lambda

Start-EC2-Instances
```

This starts the development servers every weekday at **8:00 AM**.

---

# Step 10: Create EventBridge Scheduler for Stop

Create another schedule.

Schedule Name

```
Stop-Dev-Servers
```

Cron Expression

```
cron(0 23 ? * MON-FRI *)
```

Target

```
Stop-EC2-Instances
```

This stops the development servers every weekday at **11:00 PM**.

---

# Tag-Based Filtering

The Lambda functions only manage EC2 instances with the following tags:

| Tag Key | Tag Value |
|----------|-----------|
| AutoSchedule | True |
| Environment | Dev |

Instances without these tags are ignored.

---

# Benefits of This Solution

- Serverless automation
- No EC2 instance required for cron jobs
- Lower operational cost
- Fully managed by AWS
- Easy to maintain
- Highly available
- CloudWatch logging
- IAM-based security
- Easily scalable
- Follows AWS Well-Architected best practices

---

# Folder Structure

```
project/

├── start_lambda.py
├── stop_lambda.py
├── README.md
```

---

# AWS Services Used

- Amazon EC2
- AWS Lambda
- AWS IAM
- Amazon EventBridge Scheduler
- Amazon CloudWatch Logs

---

# Author

**TamilCloudBee**

Learn Cloud & DevOps in Simple Tamil.
