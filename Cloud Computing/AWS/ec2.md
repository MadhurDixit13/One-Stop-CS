# Amazon EC2 🚀

[![AWS EC2](https://img.shields.io/badge/AWS%20EC2-Compute-orange)](https://aws.amazon.com/ec2/)
[![Docs](https://img.shields.io/badge/Docs-AWS%20EC2-blue)](https://docs.aws.amazon.com/ec2/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

> Elastic Compute Cloud (EC2) provides secure, resizable compute capacity in the cloud. Run virtual machines with fine-grained control over CPU, memory, storage, and networking.

## 🎯 Overview

EC2 lets you:
- Launch on-demand or spot instances using AMIs
- Scale with Auto Scaling groups and load balancers
- Attach persistent EBS, network with VPCs, and secure with IAM
- Automate with user data, SSM, and infrastructure as code

## 🚀 Key Features
- Instance Families: General (M), Compute (C), Memory (R), Storage (I/D), GPU (G/P)
- Storage: EBS, EFS, Instance Store; snapshots and encryption
- Networking: VPC, subnets, security groups, NACLs, ENI, EIP
- Scaling: Auto Scaling groups (ASG), Elastic Load Balancing (ALB/NLB)
- Purchase Options: On-Demand, Reserved, Savings Plans, Spot
- Security: IAM roles for EC2, KMS, SSM Session Manager, IMDSv2

## 🏗️ Reference Architecture
```
AWS Account
└── VPC (10.0.0.0/16)
    ├── Public Subnet(s)
    │   ├── ALB (Internet-facing)
    │   └── EC2 (bastion/ingress)
    ├── Private Subnet(s)
    │   └── EC2 (app)
    ├── Security Groups (stateful)
    ├── NACLs (stateless)
    ├── Route Tables, IGW, NAT GW
    └── EBS Volumes / EFS
```

## 🛠️ Getting Started

### AWS CLI
```bash
# List instances
aws ec2 describe-instances --query 'Reservations[].Instances[].{Id:InstanceId,State:State.Name,Type:InstanceType,AZ:Placement.AvailabilityZone}' --output table

# Launch an instance
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=demo}]'

# Stop/Start/Terminate
aws ec2 stop-instances --instance-ids i-xxxxxxxx
aws ec2 start-instances --instance-ids i-xxxxxxxx
aws ec2 terminate-instances --instance-ids i-xxxxxxxx
```

### Python (boto3)
```python
import boto3

ec2 = boto3.client('ec2')

# List instances
resp = ec2.describe_instances()
instances = [i for r in resp['Reservations'] for i in r['Instances']]
for inst in instances:
    print(inst['InstanceId'], inst['State']['Name'], inst['InstanceType'])

# Start/Stop
ec2.start_instances(InstanceIds=['i-xxxxxxxx'])
ec2.stop_instances(InstanceIds=['i-xxxxxxxx'])
```

## 🔐 Security Essentials
- Use IAM roles for EC2 (no long-lived keys on instances)
- Enforce IMDSv2, restrict security groups to least privilege
- Prefer SSM Session Manager over SSH; manage key pairs carefully
- Encrypt EBS by default; use KMS CMKs for sensitive data

### SSM Session Manager (no SSH)
```bash
# Start a session (requires SSM agent + IAM role with SSM permissions)
aws ssm start-session --target i-xxxxxxxx
```

## 💾 Storage
- EBS: Persistent block storage; gp3, io1/io2, st1/sc1
- EFS: Managed NFS for shared, elastic storage
- Instance Store: Ephemeral, high IOPS local NVMe (data lost on stop)

```bash
# Create and attach EBS
aws ec2 create-volume --size 20 --availability-zone us-east-1a --volume-type gp3
aws ec2 attach-volume --volume-id vol-xxxxxxxx --instance-id i-xxxxxxxx --device /dev/sdf
```

## 🌐 Networking
- Elastic IPs, multiple ENIs, security groups per interface
- PrivateLink, VPC endpoints for private AWS service access

```bash
# Allocate and associate an Elastic IP
aws ec2 allocate-address --domain vpc
aws ec2 associate-address --allocation-id eipalloc-xxxx --instance-id i-xxxxxxxx
```

## 📈 Scaling
- Use Auto Scaling groups with launch templates
- Attach ALB/NLB; target tracking or step scaling policies

```bash
# Create a launch template (simplified)
aws ec2 create-launch-template \
  --launch-template-name web-lt \
  --launch-template-data '{"ImageId":"ami-xxxxxxxx","InstanceType":"t3.micro"}'
```

## 💵 Pricing
- On-Demand for flexibility; Reserved/Savings Plans for steady-state
- Spot for cost savings with interruption tolerance

## 🔧 Best Practices
1. Use SSM for access and automation; avoid opening SSH
2. Tag everything (Name, Owner, Env, CostCenter)
3. Enable detailed monitoring and CloudWatch/CloudTrail logging
4. Patch and rotate AMIs; use launch templates
5. Right-size and schedule non-prod to stop off-hours

## 🐛 Troubleshooting
- Instance unreachable: check SGs/NACLs/route tables, SSM status
- Insufficient capacity: try different AZ/instance family
- EBS throughput: verify volume type, baseline/IO credits

## 📚 Resources
- AWS EC2: https://docs.aws.amazon.com/ec2/
- Pricing: https://aws.amazon.com/ec2/pricing/
- SSM: https://docs.aws.amazon.com/systems-manager/

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#amazon-ec2-)

</div>
