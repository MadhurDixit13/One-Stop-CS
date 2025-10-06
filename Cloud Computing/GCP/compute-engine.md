# Google Compute Engine 🖥️

[![GCP Compute Engine](https://img.shields.io/badge/GCP-Compute%20Engine-blue)](https://cloud.google.com/compute)
[![Docs](https://img.shields.io/badge/Docs-Compute%20Engine-green)](https://cloud.google.com/compute/docs)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://www.apache.org/licenses/LICENSE-2.0)

> Compute Engine provides secure and customizable virtual machines on Google Cloud with global load balancing, live migration, and strong default security.

## 🎯 Overview

Compute Engine lets you:
- Launch VMs with custom machine types and accelerators
- Use persistent disks (PD-SSD/PD-Balanced), local SSDs, and snapshots
- Network with global VPC, firewall rules, and Cloud NAT
- Scale with Managed Instance Groups (MIG) and Load Balancing

## 🚀 Key Features
- Machine Types: E2, N2/N2D, C2, M2, A2 (GPU/TPU)
- Storage: Persistent Disk, Local SSD, Snapshots, Images
- Networking: Global VPC, Subnets, Firewall, Cloud NAT, Internal LB
- Scaling: Managed instance groups, autoscaling policies
- Security: Shielded VMs, OS Login, IAM, CMEK, VPC-SC
- Pricing: Sustained/committed use discounts, preemptible/spot VMs

## 🏗️ Reference Architecture
```
GCP Project
└── VPC (10.10.0.0/16)
    ├── Subnets (per region)
    ├── Firewall Rules (ingress/egress)
    ├── Cloud NAT / Cloud Router
    ├── Instance Templates → MIG
    └── Load Balancer (HTTP(S)/TCP/UDP)
```

## 🛠️ Getting Started

### gcloud CLI
```bash
# Set defaults
gcloud config set project <PROJECT_ID>
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# List instances
gcloud compute instances list

# Create an instance
gcloud compute instances create demo-vm \
  --machine-type=e2-micro \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=http-server,https-server

# Stop/Start/Delete
gcloud compute instances stop demo-vm
gcloud compute instances start demo-vm
gcloud compute instances delete demo-vm --quiet
```

### Python (google-api-python-client)
```python
from googleapiclient import discovery
from oauth2client.client import GoogleCredentials

credentials = GoogleCredentials.get_application_default()
service = discovery.build('compute', 'v1', credentials=credentials)
project = 'your-project-id'
zone = 'us-central1-a'

# List instances
req = service.instances().list(project=project, zone=zone)
resp = req.execute()
for item in resp.get('items', []):
    print(item['name'], item['status'], item['machineType'])
```

## 🔐 Security Essentials
- Use IAM and service accounts; avoid user-managed static keys on VMs
- Enable OS Login; restrict SSH and use IAP/Cloud Identity-Aware Proxy
- Use Shielded VM defaults; enable CMEK for sensitive disks

## 💾 Storage
- Persistent Disk: Zonal/Regional; PD-SSD/PD-Balanced/PD-Extreme
- Snapshots for backup and cross-region copy

```bash
# Create a disk and attach to an instance
gcloud compute disks create data-disk --size=100GB --type=pd-ssd --zone=us-central1-a
gcloud compute instances attach-disk demo-vm --disk=data-disk --zone=us-central1-a
```

## 🌐 Networking
- Global VPC with hierarchical firewall rules
- Cloud NAT for private egress; external IPs when needed

```bash
# Open HTTP (example)
gcloud compute firewall-rules create allow-http --allow=tcp:80 --target-tags=http-server
```

## 📈 Scaling
- Managed Instance Groups with instance templates and autoscaling

```bash
# Create template and MIG
gcloud compute instance-templates create web-tpl --machine-type=e2-micro --image-family=debian-12 --image-project=debian-cloud
gcloud compute instance-groups managed create web-mig --base-instance-name=web --size=2 --template=web-tpl --zone=us-central1-a
```

## 💵 Pricing
- Sustained use and committed use discounts reduce costs
- Spot VMs for fault-tolerant workloads

## 🔧 Best Practices
1. Use OS Login/IAP; minimize public SSH
2. Tag and label resources for governance and cost
3. Use MIG + load balancing for production
4. Enable Cloud Monitoring/Logging; set up alerts
5. Right-size with Recommender; stop dev VMs off-hours

## 🐛 Troubleshooting
- SSH issues: verify OS Login, firewall, IAP, serial console
- Quotas: check regional CPU/accelerator quotas
- Disk performance: choose appropriate PD type and size

## 📚 Resources
- Compute Engine: https://cloud.google.com/compute/docs
- Pricing: https://cloud.google.com/compute/pricing
- OS Login: https://cloud.google.com/compute/docs/oslogin

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#google-compute-engine-)

</div>
