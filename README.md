<p align="center">
  <img src="https://img.shields.io/badge/AWS-Secure%20Network-orange?logo=amazon-aws&style=for-the-badge" />
  <img src="https://img.shields.io/badge/Terraform-Automation-blueviolet?logo=terraform&style=for-the-badge" />
  <img src="https://img.shields.io/badge/Zero--Trust-Architecture-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" />
</p>

# ☁️ AWS Secure Network Architecture Labs
> **Repository:** [`aws-secure-network-architecture-labs`](https://github.com/olumidetowoju/aws-secure-network-architecture-labs)

Build a **10-Day enterprise-grade AWS network** from **Test → Live**, with **Terraform**, **Zero-Trust Security**, **DevSecOps pipelines**, and **governance automation**.

---

# 📚 Course Overview

| Day | Module | Description | Lab | Diagram |
|-----|---------|--------------|------|----------|
| **[Day 1](labs/day1-foundations.md)** | Secure Network Design (Foundations) | Core AWS security concepts, segmentation, Zero Trust | [Lab 1](labs/day1-foundations.md) | [Diagram 1](diagrams/day1-foundations.mmd) |
| **[Day 2](labs/day2-test-environment.md)** | Deploying the Test Environment | Build isolated VPC, subnet, and EC2 instance for testing | [Lab 2](labs/day2-test-environment.md) | [Diagram 2](diagrams/day2-test-environment.mmd) |
| **[Day 3](labs/day3-live-environment.md)** | Live (Production) Environment + Terraform | Hardened multi-tier VPC with IaC deployment | [Lab 3](labs/day3-live-environment.md) | [Diagram 3](diagrams/day3-live-environment.mmd) |
| **[Day 4](labs/day4-bastion-host.md)** | Bastion Host + Private Access | SSH tunneling, bastion configuration, CloudWatch logging | [Lab 4](labs/day4-bastion-host.md) | [Diagram 4](diagrams/day4-bastion-host.mmd) |
| **[Day 5](labs/day5-monitoring.md)** | Monitoring, Compliance & Threat Detection | CloudTrail, Config, GuardDuty, Security Hub, Alerting | [Lab 5](labs/day5-monitoring.md) | [Diagram 5](diagrams/day5-monitoring.mmd) |
| **[Day 6](labs/day6-devsecops.md)** | DevSecOps & Auto-Remediation | Terraform pipelines, tfsec, Checkov, OPA, auto-remediation | [Lab 6](labs/day6-devsecops.md) | [Diagram 6](diagrams/day6-devsecops.mmd) |
| **[Day 7](labs/day7-data-security.md)** | Data Layer Security | RDS + KMS + Secrets Manager encryption & rotation | [Lab 7](labs/day7-data-security.md) | [Diagram 7](diagrams/day7-data-security.mmd) |
| **[Day 8](labs/day8-identity-access.md)** | Identity & Access Hardening | AWS SSO, IAM boundaries, SCPs, MFA enforcement | [Lab 8](labs/day8-identity-access.md) | [Diagram 8](diagrams/day8-identity-access.mmd) |
| **[Day 9](labs/day9-incident-response.md)** | Incident Response & Forensics | GuardDuty, Detective, Athena, Lambda quarantine | [Lab 9](labs/day9-incident-response.md) | [Diagram 9](diagrams/day9-incident-response.mmd) |
| **[Day 10](labs/day10-cost-governance.md)** | Cost Optimization & Governance | Budgets, Tag Policies, SCP guardrails, FinOps | [Lab 10](labs/day10-cost-governance.md) | [Diagram 10](diagrams/day10-cost-governance.mmd) |


# ⚙️ Quick Start

git clone https://github.com/olumidetowoju/aws-secure-network-architecture-labs.git
cd aws-secure-network-architecture-labs
cat labs/day1-foundations.md
Use Mermaid Live Editor or GitHub’s built-in renderer to visualize any .mmd diagram.

# 🧱 Technology Stack

AWS: VPC, EC2, IAM, S3, RDS, CloudTrail, Config, GuardDuty, Security Hub, KMS

IaC: Terraform (modular + remote state)

Security Tools: tfsec, Checkov, Conftest, OPA

Monitoring: CloudWatch, Athena, Detective

CI/CD: GitHub Actions

Automation: Lambda, EventBridge

Governance: AWS Organizations, SCPs, Tag Policies, Budgets

# 🔄 Living README

This is a living document — each lab and diagram is interlinked for easy navigation.
You can add future days, advanced modules, or new diagrams without changing this layout.

Future Add-ons (Optional):

Day 11 – WAF & Shield Advanced

Day 12 – Multi-Account Landing Zone

Day 13 – Compliance Automation (SOC2, ISO27001)

# 🏁 Completion

By Day 10 you will have:
✅ A fully automated Zero-Trust AWS network
✅ Secure promotion pipeline from Test → Live
✅ Monitored, compliant, cost-optimized environment
✅ Governance guardrails and DevSecOps integration

# 🔗 Resources

AWS Well-Architected Framework – Security Pillar

Terraform AWS Modules Registry

AWS Security Hub CIS Benchmarks

Mermaid Diagram Syntax Reference

© 2025 Olumide Towoju — Built for real-world secure cloud operations. ☁️

# 🧭 Folder Structure
```plaintext
aws-secure-network-architecture-labs/
├── labs/ # Markdown labs (Day 1-10)
├── diagrams/ # Mermaid diagrams (.mmd)
├── terraform/ # IaC code and automation
│ ├── environments/
│ ├── lambda/
│ └── *.tf
├── .github/workflows/ # GitHub Actions pipelines
├── anki/ # Flashcards / revision
└── README.md # Living course index

