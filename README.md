# Azure Infrastructure with Terraform

This project sets up a basic Azure infrastructure including a Linux Virtual Machine using Terraform. It was designed to troubleshoot and establish a stable development environment accessible via VS Code Remote-SSH.

## Infrastructure Components

The Terraform script (`main.tf`) provisions the following resources in the `swedencentral` region:

- **Resource Group**: `mtc-ressources`
- **Networking**:
  - Virtual Network (`mtc-network`) with address space `10.123.0.0/16`
  - Subnet (`mtc-subnet`) with prefix `10.123.1.0/24`
  - Network Security Group (`mtc-sg`) allowing all inbound traffic (Dev environment)
  - Public IP (`mtc-ip`) with Static allocation and Standard SKU
- **Compute**:
  - Network Interface (`mtc-ni`)
  - Linux Virtual Machine (`mtc-vm`) running Ubuntu 22.04 LTS

## Troubleshooting & Improvements

During the setup, we encountered and resolved several issues to ensure a stable connection:

### 1. Public IP Configuration
**Issue:** The Public IP was initially set to `Dynamic` allocation, which meant the IP address wasn't assigned until the VM was fully running, complicating the retrieval of the IP for connection.
**Fix:** Updated the `azurerm_public_ip` resource to use `Static` allocation and `Standard` SKU.

### 2. SSH Host Key Verification
**Issue:** VS Code Remote-SSH failed to connect with a "Host key verification failed" error because the new VM's fingerprint wasn't in the local `known_hosts` file.
**Fix:** Added the VM's host key to the local machine's known hosts:
```bash
ssh-keyscan -H <VM_PUBLIC_IP> >> ~/.ssh/known_hosts
```

### 3. VM Performance & Connection Timeouts
**Issue:** The SSH connection would time out or the VM would crash when VS Code Remote-SSH attempted to install its server component.
**Cause:** The initial VM size `Standard_B1s` (1 vCPU, 1 GiB RAM) was insufficient for running the VS Code Server alongside the OS.
**Fix:** Resized the VM to `Standard_B2s` (2 vCPUs, 4 GiB RAM) in `main.tf` to provide adequate resources.

## Usage

### Deploy Infrastructure
```bash
terraform init
terraform apply -auto-approve
```

### Connect via SSH
You can connect using the admin user and the SSH key configured in the script:
```bash
ssh -i ~/.ssh/mtcazurekey adminuser@<VM_PUBLIC_IP>
```

### Destroy Infrastructure
To clean up all resources:
```bash
terraform destroy -auto-approve
```
