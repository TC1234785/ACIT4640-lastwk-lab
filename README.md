# Lab Week 11 - Infrastructure Automation with Terraform and Ansible

This lab demonstrates infrastructure provisioning using Terraform and configuration management using Ansible to deploy nginx web servers on AWS EC2 instances.

## Architecture

- **VPC** with public subnet in us-west-2a
- **Two EC2 Instances:**
  - Web Server: Debian 13, tagged "Web"
  - Database Server: Rocky Linux, tagged "Database"
- **Nginx** installed on both servers with custom HTML pages
- **Dynamic Ansible Inventory** using AWS EC2 plugin

## Prerequisites

Before running this lab, ensure you have the following installed:

```bash
# Install AWS CLI
# https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

# Install Terraform
# https://developer.hashicorp.com/terraform/install

# Install Ansible
sudo apt update
sudo apt install ansible -y

# Install Python dependencies for Ansible AWS integration
pip install boto3 botocore

# Configure AWS CLI with your credentials
aws configure
```

## Step 1: Generate SSH Key Pair

Generate an SSH key pair for accessing the EC2 instances:

```bash
# Generate SSH key pair (press Enter to accept defaults)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws-4640 -C "aws-4640-key"

# Set proper permissions
chmod 600 ~/.ssh/aws-4640
chmod 644 ~/.ssh/aws-4640.pub
```

## Step 2: Upload SSH Key to AWS

Use the provided script to import your public key to AWS:

```bash
# Navigate to scripts directory
cd scripts

# Make the script executable
chmod +x import_lab_key

# Import the key to AWS
./import_lab_key ~/.ssh/aws-4640.pub

# Verify the key was imported
aws ec2 describe-key-pairs --key-names aws-4640
```

## Step 3: Provision Infrastructure with Terraform

Navigate to the terraform directory and initialize:

```bash
cd ../terraform

# Initialize Terraform
terraform init

# Format Terraform files
terraform fmt

# Validate configuration
terraform validate

# Preview changes
terraform plan

# Apply configuration (type 'yes' when prompted)
terraform apply
```

**Note the output:** Terraform will display the public IP addresses of both EC2 instances. Save these for testing.

Example output:
```
database_server = "54.x.x.x"
web_server = "34.x.x.x"
```

## Step 4: Configure Servers with Ansible

Wait 2-3 minutes for EC2 instances to fully initialize, then run Ansible:

```bash
cd ../ansible

# Test dynamic inventory
ansible-inventory -i inventory/aws_ec2.yml --graph

# Verify connectivity (may take a minute for SSH to be ready)
ansible all -i inventory/aws_ec2.yml -m ping

# Run the playbook to configure both servers
ansible-playbook -i inventory/aws_ec2.yml playbook.yml
```

The playbook will:
- Install and configure nginx on both servers
- Deploy custom HTML pages with "Hello from web" and "Hello from database"
- Start and enable nginx service

## Step 5: Validate the Setup

Access each server via HTTP in your browser:

```bash
# Web Server (Debian)
http://<web_server_public_ip>

# Database Server (Rocky Linux)
http://<database_server_public_ip>
```

You should see:
- **Web Server:** HTML page displaying "Hello from web"
- **Database Server:** HTML page displaying "Hello from database"

## Screenshots

### Web Server Screenshot
![Web Server - Hello from web](screenshots/web_server.png)

### Database Server Screenshot
![Database Server - Hello from database](screenshots/database_server.png)

*Note: Screenshots should show the browser with the public IP in the URL bar*

## Cleanup

When finished, destroy all AWS resources to avoid charges:

```bash
# Destroy Terraform-managed infrastructure
cd terraform
terraform destroy

# Delete the SSH key from AWS
cd ../scripts
./delete_lab_key
```

## Troubleshooting

### Ansible can't connect to instances
- Wait 2-3 minutes after `terraform apply` for instances to fully boot
- Verify security group allows SSH (port 22) and HTTP (port 80)
- Check that the correct user is specified: `admin` for Debian, `rocky` for Rocky Linux

### Terraform can't find Debian AMI
- Ensure you're in the us-west-2 region
- The AMI ID may change; update the data source filter if needed

### Dynamic inventory not finding instances
- Ensure boto3 and botocore are installed: `pip install boto3 botocore`
- Verify AWS credentials are configured: `aws sts get-caller-identity`
- Check that instances have the correct tags (Role: web_server or database_server)

## Project Structure

```
lab_wk_11/
├── README.md                           # This file
├── scripts/
│   ├── import_lab_key                  # Import SSH key to AWS
│   └── delete_lab_key                  # Delete SSH key from AWS
├── terraform/
│   ├── main.tf                         # Main infrastructure configuration
│   ├── provider.tf                     # AWS provider configuration
│   └── modules/
│       └── web-server/                 # EC2 instance module
├── ansible/
│   ├── ansible.cfg                     # Ansible configuration
│   ├── playbook.yml                    # Main playbook
│   ├── inventory/
│   │   └── aws_ec2.yml                # Dynamic inventory configuration
│   └── roles/
│       ├── web_server/                 # Web server role (Debian)
│       │   ├── tasks/
│       │   ├── handlers/
│       │   ├── templates/
│       │   └── files/
│       └── database_server/            # Database server role (Rocky Linux)
│           ├── tasks/
│           ├── handlers/
│           ├── templates/
│           └── files/
```

## Technologies Used

- **Terraform**: Infrastructure as Code (IaC) for AWS provisioning
- **Ansible**: Configuration management and application deployment
- **AWS EC2**: Virtual servers
- **AWS VPC**: Network isolation
- **Nginx**: Web server
- **Debian 13**: Linux distribution for web server
- **Rocky Linux**: Linux distribution for database server

## Author

Completed for ACIT 4640 - Lab Week 11

