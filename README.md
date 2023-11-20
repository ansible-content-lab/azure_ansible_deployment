# Ansible Collection - lab.azure_deployment

This is a collection that will deploy Red Hat Ansible Automation Platform on Microsoft Azure.
While this collection will work with any Ansible deployment on Azure, this is intended to be a starting point for customers that purchase Ansible Automation Platform subscriptions from the Azure marketplace.
Take this collection and enhance/improve/update based on the resources that you need for your AAP deployment.

## Introduction

This collection performs the following actions in the order listed.
There are variables that can be used to skip or prevent steps by setting variables, which are covered later in this document.

| Step | Description |
| ---- | ----------- |
| Create a Deployment ID | Creates a random string that will be used in tagging for correlating the resources used with a deployment of AAP. |
| Create a resource group | Creates a resource group to contain all of the related resources for the AAP installation. |
| Create a Virtual Network | Creates a virtual network with a CIDR block that can contain the subnets that will be created. |
| Create Subnets | Creates the subnets for automation controller, execution environments, private automation hub, and Event-Driven Ansible. |
| Create Public IPs | Creates public IPs so that the VMs have access to the internet. |
| Create a Network Security Group | Creates a security group that allows AAP ports within the VNET, and HTTPS and automation mesh ports externally. |

## Getting Started

These sections will describe required or recommended steps so that your Ansible Automation Platform deployment is as seamless as possible.

### This Collection

If you do not intend to make changes to the collection, then you can install directly from the `ansible-galaxy` CLI tool.
Examples in this readme will assume that you have done this.

```bash
ansible-galaxy collection install git+https://github.com/ansible-content-lab/azure_ansible_deployment.git
```

You may also download the collection from GitHub and modify to suit your needs.

### Local Ansible Configuration

You should also ensure that the `ansible.cfg` file on the machine where you will run the deployment playbook is configured to keep the SSH connection to the VM alive since the AAP installer process takes about 30 mins to run.
This collection includes an example `ansible.cfg` file, but your local Ansible deployment may use a different file.
Add the following to your file to ensure that waiting for the installer does not cause this collection to time out.

```ini
[ssh_connection]
ssh_args = -o ServerAliveInterval=30
```

### Roles

This collection includes the following roles.
Each role has default variables and required variables.
Review the default variables files to view all of the options that may be set.

| Role | Description |
| ---- | ----------- |
| `lab.azure_deployment.infrastructure` | Responsible for deploying the Azure infrastructure. |

### Azure Credentials

The Azure collection used as a dependency requires Azure credentials, which can be set in different places, such as the `~/.azure/credentials` file above, through environment variables, or the Azure CLI profile.
The easiest, and most portable, approach will be to set the following env vars.

- `AZURE_CLIENT_ID`
- `AZURE_SECRET`
- `AZURE_SUBSCRIPTION_ID`
- `AZURE_TENANT`

The playbooks included in this collection will need a way to connect to the virtual machines that it creates.
By default, VMs are created with public IP addresses to make this simple, but the collection may be modified to use private IP addresses if your local machine can route traffic to private networks.

## Deploying Ansible Automation Platform

This section will walk through deploying the Azure infrastructure and Ansible Automation Platform.

### Checklist

- [ ] Install this collection (or download and modify)
- [ ] Ensure that `ansible.cfg` is updated to keep SSH connections alive.
- [ ] Ansible CLI tools installed locally (`ansible`)
- [ ] Configure the Azure environment variables for authentication

### Running the Playbook

Assuming that all variables are configured properly and your AWS account has permissions to deploy the resources defined in this collection, then running the playbook should be a single task.

```bash
ansible-playbook lab.azure_deployment.deploy_aap
```

## Uninstall

The `playbooks/destroy_aap.yml` playbook will remove RHEL subscription entitlements and deprovision the infrastructure that has been associated with a deployment id.
This will permanently remove all data, so only run this playbook if you are sure that you want to delete all traces of the deployment.

```bash
ansible-playbook lab.azure_deployment.destroy_aap
```
