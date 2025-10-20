# 🧯 Day 9 – Incident Response & Forensics (AWS Detective / Athena / Automation)

## 🎯 Objective
Detect, investigate, and automatically respond to suspicious activity using AWS GuardDuty, Security Hub, Detective, Athena, and EventBridge playbooks.

---

## 🧱 1. Architecture Overview

```mermaid
graph TD
A[GuardDuty Findings] --> B[Security Hub]
B --> C[EventBridge Rules]
C --> D[Lambda Response Playbooks]
D --> E[Quarantine Instance (SG)]
B --> F[AWS Detective]
F --> G[Athena Logs Query]
G --> H[S3 CloudTrail / VPC Flow Logs]
Tool	Purpose
GuardDuty	Detect threats (anomalies, port scans, compromised IAM)
Security Hub	Centralize findings and score risk
AWS Detective	Investigate activity timeline
Athena	Search CloudTrail + VPC Flow Logs
EventBridge + Lambda	Automated containment

⚙️ 2. Enable AWS Detective & Link GuardDuty
bash
Copy code
aws detective create-graph
aws detective create-members --graph-arn <graph-arn> --account-ids <account-id>
aws guardduty enable-organization-admin-account --admin-account-id <admin-id>
🧩 3. EventBridge Rule for High Severity Findings
Create incident-response.tf in terraform:

h
Copy code
resource "aws_cloudwatch_event_rule" "high_gd" {
  name        = "guardduty-high"
  description  = "Trigger Lambda on GuardDuty High Severity"
  event_pattern = jsonencode({
    source = ["aws.guardduty"],
    detail-type = ["GuardDuty Finding"],
    detail = { severity = [{ numeric = [">=", 7] }] }
  })
}

resource "aws_lambda_function" "quarantine" {
  filename      = "lambda/quarantine.zip"
  function_name = "quarantine_instance"
  role          = aws_iam_role.lambda_ir.arn
  handler       = "index.handler"
  runtime       = "python3.12"
}

resource "aws_cloudwatch_event_target" "high_gd_target" {
  rule   = aws_cloudwatch_event_rule.high_gd.name
  arn    = aws_lambda_function.quarantine.arn
}
🪄 4. Lambda Playbook – Quarantine EC2
terraform/lambda/index.py

python
Copy code
import boto3, json
ec2 = boto3.client('ec2')

def handler(event, context):
  detail = event.get('detail', {})
  inst = detail.get('resource', {}).get('instanceDetails', {}).get('instanceId')
  if not inst: return {'status': 'no-instance'}
  sg = ec2.create_security_group(GroupName='QuarantineSG', Description='Isolated')
  ec2.modify_instance_attribute(InstanceId=inst, Groups=[sg['GroupId']])
  ec2.stop_instances(InstanceIds=[inst])
  return {'status': 'quarantined', 'instance': inst}
Zip & apply:

bash
Copy code
zip -j lambda/quarantine.zip lambda/index.py
terraform apply -var-file=environments/live.tfvars -auto-approve
🔍 5. Forensics via Athena
Query CloudTrail logs:

sql
Copy code
SELECT eventName, userIdentity.userName, sourceIPAddress, eventTime
FROM cloudtrail_logs
WHERE eventTime > date_sub('day', 1, current_timestamp)
AND eventName IN ('AuthorizeSecurityGroupIngress','RunInstances');
Export to CSV for investigation.

🧰 6. Investigate in AWS Detective
Search for Finding ID from GuardDuty.

Review “Linked Entities”: IAM user, IP, EC2, region.

Visualize timeline graph.

Mark “Resolved” in Security Hub after containment.

🧠 7. Incident Playbook
GuardDuty alert → Lambda auto-quarantine.

Athena queries → verify source activity.

Detective → map entity relationships.

Security Hub → document remediation.

CloudTrail → confirm no further access.

Restore from snapshot if needed.

📋 8. Testing
Simulate alerts using AWS sample findings:

bash
Copy code
aws guardduty create-sample-findings --detector-id <id>
Check Lambda logs for “quarantined” output.

🛡️ 9. Checklist
Control	Implemented	Evidence
GuardDuty Alerts	✅	Console → Findings
EventBridge Triggers	✅	Event metrics
Lambda Response	✅	Quarantine log
Detective Graph	✅	Timeline view
Athena Logs Query	✅	Results CSV

🧾 Day 9 Summary
✅ Automated incident response workflow
✅ Real-time containment (Lambda + EventBridge)
✅ Full forensics via Detective + Athena
✅ Improved MTTD & MTTR
✅ Security Hub integration for audit trail

🔖 Next
Day 10 – Cost Optimization & Governance (Budgets, SCPs, Organizations)
