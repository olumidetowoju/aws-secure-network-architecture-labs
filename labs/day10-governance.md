





































# 💰 Day 10 – Cost Optimization & Governance (Budgets / SCPs / Tag Policies)

## 🎯 Objective
Implement financial controls, governance frameworks, and continuous optimization to keep your secure AWS environment efficient, compliant, and auditable.

---

## 🧱 1. Architecture Blueprint

```mermaid
graph TD
A[Root Organization] --> B[OU: Production]
A --> C[OU: Test & Sandbox]
A --> D[OU: Security & Audit]
A --> E[OU: Shared Services]
B --> F[Accounts: Live, Analytics]
C --> G[Test, Dev Accounts]
A --> H[Budgets + SCPs + Tag Policies]
Control	Service	Goal
Budget Tracking	AWS Budgets	Enforce spend limits
Policy Guardrails	Service Control Policies	Prevent misconfigurations
Tagging Standards	Tag Policies	Uniform cost allocation
Reporting	Cost Explorer & Athena	Trend analysis

⚙️ 2. Organization Structure
Enable AWS Organizations and create OUs:

bash
Copy code
aws organizations create-organization --feature-set ALL
aws organizations create-organizational-unit --parent-id <root-id> --name Production
aws organizations create-organizational-unit --parent-id <root-id> --name Test
aws organizations create-organizational-unit --parent-id <root-id> --name Security
Attach SCPs later for each OU.

🧩 3. Budgets & Notifications
Terraform – budget.tf
h
Copy code
resource "aws_budgets_budget" "monthly_budget" {
  name              = "LiveBudget"
  budget_type       = "COST"
  limit_amount      = "200"
  limit_unit        = "USD"
  time_unit         = "MONTHLY"
  cost_filters      = { "LinkedAccount" = [var.live_account_id] }

  notification {
    comparison_operator = "GREATER_THAN"
    threshold           = 80
    threshold_type      = "PERCENTAGE"
    notification_type   = "ACTUAL"
    subscriber_email_addresses = ["finops@company.com"]
  }
}
🧰 4. Service Control Policies (SCP)
Deny Expensive Instance Types

json
Copy code
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "*",
      "Condition": { "StringEqualsIfExists": { "ec2:InstanceType": ["p3.16xlarge","p4d.24xlarge"] } }
    }
  ]
}
Attach to Test OU:

bash
Copy code
aws organizations create-policy --name DenyExpensiveInstances --type SERVICE_CONTROL_POLICY --content file://deny.json
aws organizations attach-policy --target-id <ou-id> --policy-id <policy-id>
🔖 5. Tag Policies for Cost Allocation
json
Copy code
{
  "tags": {
    "Project": {
      "tag_key": { "enforced_for": ["ec2:instance","s3:bucket"] },
      "tag_values": { "enforced_for": ["EC2","S3"], "allowed_values": ["Test","Live","Shared"] }
    },
    "Owner": {
      "tag_key": { "enforced_for": ["*"] },
      "tag_values": { "allowed_values": ["CloudTeam","Finance","DevOps"] }
    }
  }
}
Apply:

bash
Copy code
aws organizations create-policy --name TagEnforcement --type TAG_POLICY --content file://tags.json
aws organizations attach-policy --target-id <org-root-id> --policy-id <tag-policy-id>
📊 6. Cost Reporting & Queries
Enable Cost and Usage Report (CUR):

bash
Copy code
aws cur put-report-definition \
  --report-definition '{
    "ReportName":"SecureNetCostReport",
    "TimeUnit":"DAILY",
    "Format":"textORcsv",
    "Compression":"GZIP",
    "S3Bucket":"cost-reports-bucket",
    "AdditionalSchemaElements":["RESOURCES"]
  }'
Query with Athena:

sql
Copy code
SELECT
  line_item_resource_id AS Resource,
  sum(line_item_unblended_cost) AS Cost
FROM cost_usage_report
WHERE usage_start_date >= date_trunc('month', current_date)
GROUP BY 1
ORDER BY Cost DESC;
🔧 7. Automation & Remediation
EventBridge rule for Budget Exceeded → SNS → Slack / email.

Lambda to disable non-prod EC2 instances at night:

bash
Copy code
aws ec2 stop-instances --instance-ids $(aws ec2 describe-instances \
--filters "Name=tag:Environment,Values=Test" \
--query "Reservations[*].Instances[*].InstanceId" --output text)
🧠 8. Governance Best Practices
Enable AWS Organizations and Central Billing.

Apply SCPs at OU level, not per account.

Use tags for showback and budget tracking.

Automate off-hours shutdown.

Integrate Budgets API with Slack/SNS for alerting.

Re-evaluate spend quarterly with Cost Explorer forecasts.

🛡️ 9. Checklist
Control	Status	Evidence
Budget Alerts	✅	Email Notifications
SCPs Applied	✅	Org Console Policies
Tag Compliance	✅	Tag Policies Report
CUR + Athena	✅	Query Results
Cost Optimization Loop	✅	Quarterly review

🧾 Day 10 Summary
✅ Budgets enforced + email alerts
✅ SCP guardrails prevent costly actions
✅ Tag policies ensure governance
✅ Athena and CUR for financial visibility
✅ Automation for shutdown and alerting
✅ Full FinOps + SecOps governance closure

