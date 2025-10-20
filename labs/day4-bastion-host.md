# 🧱 Day 4 – Implementing Bastion Host + Private Access Patterns  

## 🎯 Objective  
Create a **secure jump-host architecture** for administrative access to private subnets, enforce **Zero Trust SSH tunneling**, and centralize session logging.  

---

## 🧠 1. Why a Bastion Host?  
A Bastion Host acts as a **controlled gateway** to your private instances.  
- Only trusted IPs can reach it.  
- No other servers are directly reachable from the Internet.  
- All admin traffic is audited and time-limited.  

---

## 🏗️ 2. Architecture Overview  

```mermaid
graph TD  
A[Public Subnet 10.1.2.0/24] --> B[Bastion Host EC2]  
A --> C[NAT Gateway]  
D[Private Subnet 10.1.1.0/24] --> E[App Server EC2]  
B -- SSH Tunnel --> E  
Security Principles:

Bastion Host = only entry point for SSH.

App servers have no public IP.

Strict IAM and Security Group controls.

⚙️ 3. Terraform Module – Bastion Host
Add to ~/secure-network-course/terraform/main.tf after the Live infrastructure module:

hcl
Copy code
resource "aws_security_group" "bastion_sg" {
  name        = "${var.env_name}-bastion-sg"
  description = "SSH access from admin IP"
  vpc_id      = module.vpc.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.admin_ip]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${var.env_name}-bastion-sg" }
}

resource "aws_instance" "bastion" {
  ami                    = "ami-0c02fb55956c7d316"
  instance_type          = "t3.micro"
  subnet_id              = element(module.vpc.public_subnets, 0)
  vpc_security_group_ids = [aws_security_group.bastion_sg.id]
  key_name               = var.key_name
  associate_public_ip_address = true
  tags = { Name = "${var.env_name}-bastion" }
}
🔧 4. Add Variable to variables.tf
hcl
Copy code
variable "admin_ip" { description = "Public IP of admin machine" }
Example in live.tfvars:

h
Copy code
admin_ip = "YOUR.PUBLIC.IP.ADDR/32"
🔐 5. Security Groups for Private Instances
Add to main.tf:

h
Copy code
resource "aws_security_group" "private_sg" {
  name        = "${var.env_name}-private-sg"
  description = "Allow SSH from Bastion only"
  vpc_id      = module.vpc.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    security_groups = [aws_security_group.bastion_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${var.env_name}-private-sg" }
}
💻 6. Deploy Private App Server
hcl
Copy code
resource "aws_instance" "private_app" {
  ami                    = "ami-0c02fb55956c7d316"
  instance_type          = "t3.micro"
  subnet_id              = element(module.vpc.private_subnets, 0)
  vpc_security_group_ids = [aws_security_group.private_sg.id]
  key_name               = var.key_name
  associate_public_ip_address = false
  tags = { Name = "${var.env_name}-private-app" }
}
Deploy changes:

bash
Copy code
cd ~/secure-network-course/terraform
terraform plan -var-file=environments/live.tfvars
terraform apply -var-file=environments/live.tfvars -auto-approve
🧩 7. SSH Tunneling Workflow
SSH into Bastion:

bash
Copy code
ssh -i SecureKey.pem ubuntu@<bastion-public-ip>
From Bastion, connect to Private App:

bash
Copy code
ssh -i SecureKey.pem ubuntu@<private-ip>
Or create local tunnel from admin machine:

bash
Copy code
ssh -i SecureKey.pem -L 8080:<private-ip>:80 ubuntu@<bastion-public-ip>
curl http://localhost:8080
📜 8. Logging and Auditing
a. Enable Session Logging:

bash
Copy code
sudo apt install auditd -w /var/log/auth.log -k ssh_login
b. Push Logs to CloudWatch:

bash
Copy code
sudo apt install -c awslogs
sudo systemctl enable awslogs --now
c. GuardDuty Findings:

Detect SSH brute force.

Detect port scanning on Bastion.

🛡️ 9. Security Checklist
Control	Implemented	Verified
Bastion restricted SSH	✅	Inbound only from admin IP
Private EC2 no public IP	✅	Ping test fails from Internet
Logs to CloudWatch	✅	Visible in /aws/bastion-logs
GuardDuty Enabled	✅	Findings monitored

🧠 10. Zero Trust Enhancements
Rotate Bastion key pairs every 90 days.

Use AWS Systems Manager Session Manager instead of SSH for final lockdown.

Disable password auth (PasswordAuthentication no).

Monitor CloudTrail events for SSH activity.

🧾 Day 4 Summary
✅ Created a secure Bastion Host for private access
✅ Restricted admin entry to verified IPs
✅ Isolated application instances in private subnets
✅ Enabled SSH tunneling + logging + auditing
✅ Advanced toward Zero Trust Network Access (ZTNA)

🔖 Next Step
Proceed to Day 5 – Monitoring & Compliance (CloudTrail, GuardDuty, Config)
