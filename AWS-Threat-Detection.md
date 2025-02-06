# AWS AI-Powered Threat Detection Cookbook

## Overview
This guide demonstrates how to use AWS AI-powered threat detection to identify security anomalies and trigger automated responses.

## Workflow
1. **Threat detection** using Amazon GuardDuty.
2. **Event-driven automation** with AWS Lambda.
3. **Alerts and notifications** via Amazon SNS.
4. **Incident management** in AWS Security Hub.

## Architecture
1. **Amazon GuardDuty** detects suspicious activities (e.g., port scanning, compromised credentials, unusual API calls).
2. **AWS EventBridge** captures GuardDuty findings and triggers an AWS Lambda function.
3. **AWS Lambda** analyzes the findings and takes actions (e.g., isolate an EC2 instance, block an IP, disable compromised credentials).
4. **Amazon SNS** sends a notification (email/SMS) to security teams.
5. **AWS Security Hub** aggregates findings for further analysis.

---

## Step-by-Step Implementation

### Step 1: Enable Amazon GuardDuty
1. Open **AWS Console** → GuardDuty.
2. Click **Enable GuardDuty** (if not already enabled).
3. Ensure findings are generated (e.g., simulate a threat using an AWS GuardDuty test event).

### Step 2: Create an EventBridge Rule to Trigger Lambda
1. Navigate to **Amazon EventBridge** → Rules → Create Rule.
2. Name the rule: `GuardDuty-Threat-Response`.
3. Select **Event Pattern** and choose **GuardDuty Findings** as the event source.
4. Choose **AWS Lambda** as the target and select (or create) a Lambda function.

### Step 3: Create a Lambda Function for Automated Response
This Python Lambda function isolates an EC2 instance if a threat is detected:

```python
import boto3
import json

ec2_client = boto3.client("ec2")

def lambda_handler(event, context):
    print("Received event: " + json.dumps(event, indent=2))
    
    finding = event['detail']
    instance_id = finding.get("resource", {}).get("instanceDetails", {}).get("instanceId")
    
    if instance_id:
        print(f"Isolating instance: {instance_id}")
        ec2_client.modify_instance_attribute(InstanceId=instance_id, Groups=[])
        
    return {
        'statusCode': 200,
        'body': json.dumps('Instance isolated successfully!')
    }
```

🔹 This function removes all security groups from the compromised instance, isolating it from the network.

### Step 4: Configure SNS for Notifications
1. Go to **Amazon SNS** → Create Topic (e.g., `ThreatAlerts`).
2. Create a **subscription** (email or SMS) to receive alerts.
3. Modify the Lambda function to publish notifications:

```python
sns_client = boto3.client("sns")

def send_alert(message):
    response = sns_client.publish(
        TopicArn="arn:aws:sns:region:account-id:ThreatAlerts",
        Message=message,
        Subject="Security Alert: AWS Threat Detected"
    )
    print("Alert sent:", response)
```

### Step 5: Monitor Findings in AWS Security Hub
1. Enable **Security Hub** from AWS Console.
2. Configure **integrations** to ingest GuardDuty findings.
3. **Visualize threats** and prioritize responses.

---

## Demo Execution
1. Simulate a security event (e.g., unauthorized API calls, port scanning).
2. Observe how **GuardDuty detects** the issue.
3. **EventBridge triggers Lambda**, which isolates the instance.
4. **SNS sends a notification** to the security team.
5. **Findings are aggregated** in AWS Security Hub for further analysis.

---

## Conclusion
By following this guide, you can set up a robust AWS AI-powered threat detection system that automates response actions, alerts security teams, and centralizes threat management. 
