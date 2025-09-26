# Terraform 🏗️

[![Terraform](https://img.shields.io/badge/Terraform-Infrastructure-purple)](https://terraform.io/)
[![Go](https://img.shields.io/badge/Go-1.19+-blue)](https://golang.org/)
[![License](https://img.shields.io/badge/License-MPL%202.0-green.svg)](https://opensource.org/licenses/MPL-2.0)

> **Infrastructure as Code** - Terraform is an open-source infrastructure as code software tool that enables you to safely and predictably create, change, and improve infrastructure using a declarative configuration language.

## 🎯 Overview

Terraform is a powerful infrastructure management tool that:
- **Manages Infrastructure** - Create, update, and destroy cloud resources
- **Uses Declarative Syntax** - Describe desired state, not steps
- **Supports Multiple Providers** - AWS, Azure, GCP, and 1000+ providers
- **Enables Collaboration** - Team-based infrastructure management
- **Provides State Management** - Track infrastructure changes
- **Ensures Consistency** - Reproducible infrastructure deployments

## 🚀 Key Features

### Core Capabilities
- **Infrastructure as Code** - Version control your infrastructure
- **Multi-Cloud Support** - Manage resources across different cloud providers
- **State Management** - Track infrastructure state and changes
- **Dependency Management** - Automatically handle resource dependencies
- **Plan and Apply** - Preview changes before applying them
- **Modularity** - Reusable infrastructure components

### Provider Ecosystem
- **Cloud Providers** - AWS, Azure, Google Cloud, DigitalOcean
- **Infrastructure** - Docker, Kubernetes, VMware, OpenStack
- **Databases** - MySQL, PostgreSQL, MongoDB, Redis
- **Monitoring** - Prometheus, Grafana, DataDog, New Relic
- **Security** - Vault, Consul, Keycloak

## 🏗️ Architecture

### Core Components
```
Terraform Architecture
├── 📝 Configuration Files
│   ├── .tf Files (HCL)
│   ├── Variables
│   └── Outputs
├── 🔧 Terraform Core
│   ├── Configuration Parser
│   ├── State Manager
│   └── Provider Interface
├── 🔌 Providers
│   ├── AWS Provider
│   ├── Azure Provider
│   └── Custom Providers
└── 💾 State Backend
    ├── Local State
    ├── Remote State
    └── State Locking
```

## 🛠️ Installation & Setup

### Binary Installation
```bash
# Download Terraform
wget https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
unzip terraform_1.5.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify installation
terraform version
```

### Package Manager Installation
```bash
# Ubuntu/Debian
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform

# macOS with Homebrew
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Windows with Chocolatey
choco install terraform
```

### Docker Installation
```bash
# Run Terraform in Docker
docker run --rm -v $(pwd):/workspace -w /workspace hashicorp/terraform:latest init
docker run --rm -v $(pwd):/workspace -w /workspace hashicorp/terraform:latest plan
docker run --rm -v $(pwd):/workspace -w /workspace hashicorp/terraform:latest apply
```

## 📝 Basic Configuration

### Simple AWS EC2 Instance
```hcl
# main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "HelloWorld"
  }
}
```

### Variables and Outputs
```hcl
# variables.tf
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# outputs.tf
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.web.public_ip
}

output "public_dns" {
  description = "Public DNS name of the EC2 instance"
  value       = aws_instance.web.public_dns
}
```

## 🏗️ Advanced Configuration

### Modules
```hcl
# modules/ec2/main.tf
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  vpc_security_group_ids = var.security_group_ids
  
  tags = merge(var.tags, {
    Name = var.name
  })
}

# modules/ec2/variables.tf
variable "ami" {
  description = "AMI ID"
  type        = string
}

variable "instance_type" {
  description = "Instance type"
  type        = string
}

variable "subnet_id" {
  description = "Subnet ID"
  type        = string
}

variable "security_group_ids" {
  description = "Security group IDs"
  type        = list(string)
}

variable "name" {
  description = "Instance name"
  type        = string
}

variable "tags" {
  description = "Additional tags"
  type        = map(string)
  default     = {}
}

# modules/ec2/outputs.tf
output "instance_id" {
  description = "Instance ID"
  value       = aws_instance.this.id
}

output "private_ip" {
  description = "Private IP address"
  value       = aws_instance.this.private_ip
}

output "public_ip" {
  description = "Public IP address"
  value       = aws_instance.this.public_ip
}
```

### Using Modules
```hcl
# main.tf
module "web_server" {
  source = "./modules/ec2"
  
  ami                    = "ami-0c55b159cbfafe1d0"
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public.id
  security_group_ids     = [aws_security_group.web.id]
  name                   = "web-server"
  
  tags = {
    Environment = "production"
    Project     = "web-app"
  }
}

module "database_server" {
  source = "./modules/ec2"
  
  ami                    = "ami-0c55b159cbfafe1d0"
  instance_type          = "t3.small"
  subnet_id              = aws_subnet.private.id
  security_group_ids     = [aws_security_group.database.id]
  name                   = "database-server"
  
  tags = {
    Environment = "production"
    Project     = "web-app"
  }
}
```

## ☁️ Cloud Provider Examples

### AWS Infrastructure
```hcl
# aws-infrastructure.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.project_name}-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.project_name}-igw"
  }
}

# Public Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.project_name}-public-subnet"
  }
}

# Private Subnet
resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = data.aws_availability_zones.available.names[1]
  
  tags = {
    Name = "${var.project_name}-private-subnet"
  }
}

# Security Group
resource "aws_security_group" "web" {
  name_prefix = "${var.project_name}-web-"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${var.project_name}-web-sg"
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = file("${path.module}/user_data.sh")
  
  tags = {
    Name = "${var.project_name}-web-server"
  }
}

# Data sources
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}
```

### Azure Infrastructure
```hcl
# azure-infrastructure.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = "${var.project_name}-rg"
  location = var.azure_region
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "${var.project_name}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

# Subnet
resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

# Network Security Group
resource "azurerm_network_security_group" "main" {
  name                = "${var.project_name}-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  
  security_rule {
    name                       = "SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

# Virtual Machine
resource "azurerm_linux_virtual_machine" "main" {
  name                = "${var.project_name}-vm"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "adminuser"
  
  disable_password_authentication = true
  
  network_interface_ids = [azurerm_network_interface.main.id]
  
  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}
```

## 🔧 State Management

### Local State
```hcl
# terraform.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### Remote State (S3)
```hcl
# terraform.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "production/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### State Locking
```hcl
# DynamoDB table for state locking
resource "aws_dynamodb_table" "terraform_locks" {
  name           = "terraform-locks"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
  
  tags = {
    Name = "Terraform State Locking"
  }
}
```

## 🔧 Workspaces

### Workspace Management
```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new staging

# Select workspace
terraform workspace select production

# Show current workspace
terraform workspace show
```

### Workspace-Specific Configuration
```hcl
# variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

# main.tf
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = var.environment == "prod" ? "t3.medium" : "t2.micro"
  
  tags = {
    Name        = "web-server"
    Environment = var.environment
  }
}
```

## 🔧 Best Practices

### File Organization
```
terraform-project/
├── main.tf                 # Main configuration
├── variables.tf            # Input variables
├── outputs.tf              # Output values
├── terraform.tf            # Provider and backend config
├── terraform.tfvars        # Variable values
├── terraform.tfvars.example # Example variable values
├── modules/                # Reusable modules
│   ├── ec2/
│   ├── rds/
│   └── vpc/
├── environments/           # Environment-specific configs
│   ├── dev/
│   ├── staging/
│   └── prod/
└── scripts/                # Helper scripts
    ├── deploy.sh
    └── destroy.sh
```

### Naming Conventions
```hcl
# Use consistent naming
resource "aws_instance" "web_server" {
  # Use descriptive names
}

resource "aws_security_group" "web_sg" {
  # Use prefixes for grouping
}

# Use tags consistently
tags = {
  Name        = "web-server"
  Environment = "production"
  Project     = "web-app"
  Owner       = "team-name"
  CostCenter  = "engineering"
}
```

### Security Best Practices
```hcl
# Use data sources for sensitive information
data "aws_ssm_parameter" "db_password" {
  name = "/myapp/database/password"
}

# Use least privilege IAM roles
resource "aws_iam_role" "ec2_role" {
  name = "ec2-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

# Use security groups with minimal access
resource "aws_security_group" "web" {
  name_prefix = "web-"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"] # Only internal traffic
  }
}
```

## 🚀 Advanced Features

### Data Sources
```hcl
# Get current AWS account ID
data "aws_caller_identity" "current" {}

# Get availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Get AMI ID
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Use data sources
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-server"
    AccountID = data.aws_caller_identity.current.account_id
  }
}
```

### Local Values
```hcl
# locals.tf
locals {
  common_tags = {
    Project     = "web-app"
    Environment = var.environment
    Owner       = "team-name"
    ManagedBy   = "terraform"
  }
  
  instance_count = var.environment == "prod" ? 3 : 1
  
  availability_zones = data.aws_availability_zones.available.names
}

# Use locals
resource "aws_instance" "web" {
  count         = local.instance_count
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  
  tags = merge(local.common_tags, {
    Name = "web-server-${count.index + 1}"
  })
}
```

### Dynamic Blocks
```hcl
# Dynamic security group rules
resource "aws_security_group" "web" {
  name_prefix = "web-"
  
  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}

# Variables
variable "allowed_ports" {
  description = "List of allowed ports"
  type        = list(number)
  default     = [80, 443, 8080]
}
```

## 🔧 CI/CD Integration

### GitHub Actions
```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.0
    
    - name: Terraform Format
      run: terraform fmt -check
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Validate
      run: terraform validate
    
    - name: Terraform Plan
      run: terraform plan
      env:
        TF_VAR_aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
        TF_VAR_aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve
      env:
        TF_VAR_aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
        TF_VAR_aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### GitLab CI
```yaml
# .gitlab-ci.yml
stages:
  - validate
  - plan
  - apply

variables:
  TF_ROOT: ${CI_PROJECT_DIR}
  TF_ADDRESS: ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/default

before_script:
  - cd ${TF_ROOT}
  - terraform --version
  - terraform init

validate:
  stage: validate
  script:
    - terraform validate
    - terraform fmt -check

plan:
  stage: plan
  script:
    - terraform plan
  artifacts:
    paths:
      - ${TF_ROOT}/plan.cache
    reports:
      terraform: ${TF_ROOT}/plan.json

apply:
  stage: apply
  script:
    - terraform apply -auto-approve
  when: manual
  only:
    - main
```

## 🐛 Troubleshooting

### Common Issues
1. **State Lock Issues** - Check for concurrent operations
2. **Provider Version Conflicts** - Update provider versions
3. **Resource Dependencies** - Use explicit dependencies
4. **Variable Validation** - Check variable constraints
5. **Permission Issues** - Verify IAM permissions

### Debugging Commands
```bash
# Enable debug logging
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log

# Check configuration
terraform validate

# Show current state
terraform show

# List resources
terraform state list

# Import existing resources
terraform import aws_instance.web i-1234567890abcdef0

# Refresh state
terraform refresh

# Plan with detailed output
terraform plan -detailed-exitcode
```

## 📚 Learning Resources

### Official Documentation
- [Terraform Documentation](https://terraform.io/docs/)
- [Provider Documentation](https://registry.terraform.io/)
- [CLI Commands](https://terraform.io/docs/cli/commands/)

### Community Resources
- [GitHub Repository](https://github.com/hashicorp/terraform)
- [Community Forum](https://discuss.hashicorp.com/c/terraform-core)
- [Terraform Registry](https://registry.terraform.io/)

## 🔄 Updates and Roadmap

### Recent Updates
- **v1.5.0** - Enhanced configuration language and improved performance
- **v1.4.0** - Better error handling and debugging capabilities
- **v1.3.0** - Improved state management and workspace features

### Upcoming Features
- **v1.6.0** - Enhanced provider ecosystem
- **v1.7.0** - Improved debugging and troubleshooting tools
- **v1.8.0** - Advanced state management features

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow Go coding standards
- Write comprehensive tests
- Update documentation
- Ensure backward compatibility

## 📄 License

This project is licensed under the Mozilla Public License 2.0 - see the [LICENSE](https://github.com/hashicorp/terraform/blob/main/LICENSE) file for details.

## 🙏 Acknowledgments

- **HashiCorp** for creating and maintaining Terraform
- **Open Source Community** for contributions and feedback
- **Cloud Providers** for supporting the ecosystem
- **Users** for providing real-world use cases and improvements

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#terraform-)

</div>

