# 🚀 Tamil CloudBee AWS Lambda Demo

Automatically **start** and **stop** Amazon EC2 instances based on tags using **AWS Lambda**, **Amazon EventBridge Scheduler**, and **IAM**.

This project demonstrates a simple, serverless approach to automate EC2 instance scheduling without maintaining cron servers.

---

# 📌 Architecture

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

# 🛠 AWS Services Used

- Amazon EC2
- AWS Lambda
- AWS IAM
- Amazon EventBridge Scheduler
- Amazon CloudWatch Logs

---

# 📋 Prerequisites

- AWS Account
- Amazon EC2 Instances
- IAM permissions to create:
  - IAM Policies
  - IAM Roles
  - Lambda Functions
  - EventBridge Schedulers

---

# 🏷 Step 1: Tag Your EC2 Instances

Tag every EC2 instance that should be automatically managed.

| Tag Key | Tag Value |
|----------|-----------|
| AutoSchedule | True |
| Environment | Dev |

Example

```
AutoSchedule = True
Environment = Dev
```

> **Note:** Only EC2 instances containing **both** tags will be managed.

---

# 🔐 Step 2: Create IAM Policy for Start Lambda

Navigate to

```
IAM
→ Policies
→ Create Policy
```

Use the following policy.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StartInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

Policy Name

```
Lambda-Start-EC2-Policy
```

---

# 🔐 Step 3: Create IAM Policy for Stop Lambda

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

Policy Name

```
Lambda-Stop-EC2-Policy
```

---

# 👤 Step 4: Create IAM Role for Start Lambda

Navigate to

```
IAM
→ Roles
→ Create Role
```

Trusted Entity

- AWS Service

Use Case

- Lambda

Attach Policies

- AWSLambdaBasicExecutionRole
- Lambda-Start-EC2-Policy

Role Name

```
Lambda-Start-EC2-Role
```

---

# 👤 Step 5: Create IAM Role for Stop Lambda

Attach Policies

- AWSLambdaBasicExecutionRole
- Lambda-Stop-EC2-Policy

Role Name

```
Lambda-Stop-EC2-Role
```

---

# 🚀 Step 6: Create Start Lambda Function

Navigate to

```
AWS Lambda
→ Create Function
```

Function Name

```
Start-EC2-Instances
```

Runtime

```
Python 3.13
```

Execution Role

```
Lambda-Start-EC2-Role
```

Deploy the following code.

```python
import boto3

ec2 = boto3.client("ec2")


def lambda_handler(event, context):

    response = ec2.describe_instances(
        Filters=[
            {
                "Name": "tag:AutoSchedule",
                "Values": ["True"]
            },
            {
                "Name": "tag:Environment",
                "Values": ["Dev"]
            },
            {
                "Name": "instance-state-name",
                "Values": ["stopped"]
            }
        ]
    )

    instance_ids = []

    for reservation in response["Reservations"]:
        for instance in reservation["Instances"]:
            instance_ids.append(instance["InstanceId"])

    if not instance_ids:
        print("No stopped instances found.")
        return

    ec2.start_instances(InstanceIds=instance_ids)

    print(f"Started instances: {instance_ids}")
```

Deploy the function.

---

# 🛑 Step 7: Create Stop Lambda Function

Function Name

```
Stop-EC2-Instances
```

Runtime

```
Python 3.13
```

Execution Role

```
Lambda-Stop-EC2-Role
```

Deploy the following code.

```python
import boto3

ec2 = boto3.client("ec2")


def lambda_handler(event, context):

    response = ec2.describe_instances(
        Filters=[
            {
                "Name": "tag:AutoSchedule",
                "Values": ["True"]
            },
            {
                "Name": "tag:Environment",
                "Values": ["Dev"]
            },
            {
                "Name": "instance-state-name",
                "Values": ["running"]
            }
        ]
    )

    instance_ids = []

    for reservation in response["Reservations"]:
        for instance in reservation["Instances"]:
            instance_ids.append(instance["InstanceId"])

    if not instance_ids:
        print("No running instances found.")
        return

    ec2.stop_instances(InstanceIds=instance_ids)

    print(f"Stopped instances: {instance_ids}")
```

Deploy the function.

---

# ✅ Step 8: Test the Lambda Functions

From the Lambda Console

Click

```
Test
```

Verify

- Start Lambda starts stopped instances.
- Stop Lambda stops running instances.
- Review CloudWatch Logs for execution details.

---

# ⏰ Step 9: Create EventBridge Scheduler (Start)

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

Cron Expression (Weekdays 8:00 AM UTC)

```
cron(0 8 ? * MON-FRI *)
```

Target

```
Start-EC2-Instances
```

---

# 🌙 Step 10: Create EventBridge Scheduler (Stop)

Schedule Name

```
Stop-Dev-Servers
```

Cron Expression (Weekdays 11:00 PM UTC)

```
cron(0 23 ? * MON-FRI *)
```

Target

```
Stop-EC2-Instances
```

---

# 🏷 Tag-Based Filtering

Only EC2 instances with both tags below are managed.

| Tag Key | Tag Value |
|----------|-----------|
| AutoSchedule | True |
| Environment | Dev |

Instances without these tags are ignored.

---

# 📁 Project Structure

```
project/

├── start_lambda.py
├── stop_lambda.py
└── README.md
```

---

# 🎯 Benefits

- ✅ Serverless Automation
- ✅ No Cron EC2 Server Required
- ✅ Lower AWS Cost
- ✅ Fully Managed
- ✅ Easy to Maintain
- ✅ Highly Available
- ✅ CloudWatch Logging
- ✅ IAM-Based Security
- ✅ Scalable
- ✅ AWS Well-Architected Best Practices

---

# 📖 Future Enhancements

- Support multiple environments (Dev, QA, UAT, Prod)
- Read schedules from AWS Systems Manager Parameter Store
- Send notifications using Amazon SNS
- Add Slack or Microsoft Teams notifications
- Start/Stop based on AWS Resource Groups
- Support Auto Scaling Groups
- Multi-region scheduling

---

# 👨‍💻 Author

## Tamil CloudBee

**Learn Cloud, DevOps, Linux & AI in Simple Tamil**

If this project helped you, please ⭐ Star the repository and consider sharing it with others!

---

## ⭐ Support the Channel

If you found this project useful,

- 👍 Like
- 💬 Comment
- 🔄 Share
- ⭐ Star this Repository
- ▶️ Subscribe to **Tamil CloudBee**

Happy Learning! 🚀
