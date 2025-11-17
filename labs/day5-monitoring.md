# 🛡️ Day 5 – Monitoring, Compliance & Threat Detection  

## 🎯 Objective  
Establish full visibility, compliance enforcement, and automated threat detection using **AWS CloudTrail**, **Config**, **GuardDuty**, and **CloudWatch**.  

---

## 🧱 1. Why Monitoring Matters  

Without monitoring:  
- Attacks go unnoticed.  
- Compliance violations accumulate.  
- Incidents can’t be reconstructed.  

With proper AWS monitoring, every API call, flow, and configuration change is auditable and traceable — the backbone of **Zero Trust accountability**.  

---

## ⚙️ 2. Core Components Overview  

| Service | Purpose | Key Output |  
|----------|----------|-------------|  
| **CloudTrail** | Tracks all API actions | Logs to S3 & CloudWatch |  
| **AWS Config** | Audits resource compliance | Rules + Remediation |  
| **GuardDuty** | Continuous threat detection | Findings dashboard |  
| **CloudWatch** | Logs + Metrics | Alerts & dashboards |  
| **Security Hub** | Unified compliance + findings | CIS/NIST dashboards |  

---

## 🧩 3. Terraform for Monitoring Stack  

Create a new file:  

cd ~/secure-network-course/terraform
nano monitoring.tf
Paste:

# CloudTrail
resource "aws_cloudtrail" "live_trail" {
  name                          = "LiveTrail"
  s3_bucket_name                = "live-trail-logs-${random_id.bucket_id.hex}"
  include_global_service_events = true
  is_multi_region_trail         = true
  enable_log_file_validation    = true
  is_organization_trail         = false
  enable_logging                = true
}

resource "random_id" "bucket_id" {
  byte_length = 4
}

resource "aws_s3_bucket" "trail_logs" {
  bucket = "live-trail-logs-${random_id.bucket_id.hex}"

  acl    = "private"

  versioning { enabled = true }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }

  tags = { Name = "CloudTrail-Logs" }
}

# AWS Config
resource "aws_config_configuration_recorder" "recorder" {
  name     = "config-recorder"
  role_arn = aws_iam_role.config_role.arn
}

resource "aws_config_delivery_channel" "delivery" {
  name           = "config-delivery"
  s3_bucket_name = aws_s3_bucket.trail_logs.bucket
}

resource "aws_iam_role" "config_role" {
  name = "AWSConfigRole"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = { Service = "config.amazonaws.com" }
        Action    = "sts:AssumeRole"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "config_attach" {
  role       = aws_iam_role.config_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSConfigRole"
}

# GuardDuty
resource "aws_guardduty_detector" "gd" {
  enable = true
}

## 🚀 4. Deploy Monitoring Stack

terraform init
terraform plan -var-file=environments/live.tfvars
terraform apply -var-file=environments/live.tfvars -auto-approve

## 📊 5. Enable Security Hub (Compliance Center)

aws securityhub enable-security-hub --enable-default-standards
aws securityhub get-enabled-standards
Standards automatically applied:

CIS AWS Foundations Benchmark

NIST CSF

AWS Foundational Security Best Practices

## 🔎 6. Validate CloudTrail Logging

aws cloudtrail describe-trails
aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=<your-username>
Confirm logs exist in your S3 bucket:

aws s3 ls s3://live-trail-logs-<bucket-id>/

## ⚠️ 7. Simulate Threat Scenarios

a. Unapproved SSH Attempt (GuardDuty)
Try connecting with a wrong key to trigger detection.

b. Public S3 Bucket Creation
Create one intentionally:

aws s3api create-bucket --bucket test-public-bucket --region us-east-1
aws s3api put-bucket-acl --bucket test-public-bucket --acl public-read
GuardDuty + Config should flag this as non-compliant.

Clean up:

aws s3 rb s3://test-public-bucket --force

## 📢 8. Alerts & Automation
Use CloudWatch Alarms for GuardDuty and Config events:

aws cloudwatch put-metric-alarm \
  --alarm-name GuardDutyHighSeverity \
  --metric-name FindingSeverity \
  --namespace AWS/GuardDuty \
  --statistic Maximum \
  --period 300 \
  --threshold 7 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions <sns-topic-arn>

## 🧰 9. Integrate with SNS for Notifications

aws sns create-topic --name SecurityAlerts
aws sns subscribe --topic-arn <arn> --protocol email --notification-endpoint <your-email>
You’ll receive alerts for any GuardDuty high-severity finding.

## ✅ 10. Compliance Dashboard Checks

In AWS Console:

CloudTrail: check “Trails” → “Event history”.

AWS Config: ensure recorder + delivery active.

GuardDuty: view “Findings”.

Security Hub: view CIS Benchmark report.

## 🧾 Day 5 Summary

✅ Enabled CloudTrail for API visibility
✅ Enabled AWS Config for continuous compliance
✅ Activated GuardDuty for threat detection
✅ Deployed Security Hub for compliance standards
✅ Integrated CloudWatch and SNS for alerting
✅ Completed Zero-Trust Visibility & Compliance Framework

## 🔖 Next Step

Proceed to Day 6 – Automated Remediation & DevSecOps Integration (Final Stage)
