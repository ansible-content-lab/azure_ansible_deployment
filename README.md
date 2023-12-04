# Ansible Collection - lab.azure_deployment

This is a collection that will deploy Red Hat Ansible Automation Platform on Microsoft Azure.
While this collection will work with any Ansible deployment on Azure, this is intended to be a starting point for customers that purchase Ansible Automation Platform subscriptions from the Azure marketplace.
Take this collection and enhance/improve/update based on the resources that you need for your AAP deployment.

**NOTE :** It is assumed that only one dedicated resource group will be made, used, and deleted by the playbooks and roles in this repository.

## Introduction

This collection performs the following actions in the order listed.
There are variables that can be used to skip or prevent steps by setting variables, which are covered later in this document.

| Step | Description |
| ---- | ----------- |
| Create a deployment ID | Creates a random string that will be used in tagging for correlating the resources used with a deployment of AAP. |
| Merge tags | Merges customer provided tags with the default ones. |
| Create a resource group | Creates a resource group to contain all of the related resources for the AAP installation. |
| Create a virtual network | Creates a virtual network with a CIDR block that can contain the subnets that will be created. |
| Create subnets | Creates the subnets for automation controller, execution environments, private automation hub, and Event-Driven Ansible. |
| Create the private DNS zone | Creates the private DNS zone for PostgreSQL. |
| Create a network security group | Creates a security group that allows AAP ports within the VNET and HTTPS and automation mesh ports externally. |
| Create a database server | Creates a PostgreSQL Flexible Server and the necessary databases inside of it for the controller, hub, and Event-Driven Ansible components. |
| Create the controller VMs | Creates VMs for controller, a public IP, and the virtual network interface card with the public IP attached. |
| Create the execution nodes VMs | Creates VMs for execution nodes (if enabled), a public IP, and the virtual network interface card with the public IP attached. |
| Create the hub VMs | Creates VMs for private automation hub, a public IP, and the virtual network interface card with the public IP attached. |
| Create the Event-Driven Ansible VMs | Creates VMs for Event-Driven Ansible (if enabled), a public IP, and the virtual network interface card with the public IP attached. |
| Register the VMs with Red Hat | Uses RHEL subscription manager to register each virtual machine for required RPM repos. |
| Update the VMs | Updates each VM deployed with latest kernel and packages. |
| Setup one controller VM as the installer | Configures the installer VMs with a private SSH key so that it can communicate with the other VMs that are part of the installation process and configures the installer inventory file based on the VMs that were created as part of this process. |

## Getting Started

These sections will describe required or recommended steps so that your Ansible Automation Platform deployment is as seamless as possible.

### This Collection

If you do not intend to make changes to the collection, then you can install directly from the `ansible-galaxy` CLI tool.
Examples in this readme will assume that you have done this.

```bash
ansible-galaxy collection install git+https://github.com/ansible-content-lab/azure_ansible_deployment.git
```

You may also download the collection from GitHub and modify to suit your needs.

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
- [ ] Ansible CLI tools installed locally (`ansible`)
- [ ] Configure the Azure environment variables for authentication
- [ ] Ensure you don't have anything else in the resource group that you use (default of specified via an extra var)

### Running the Playbook

The variables below are required for running the playbook

| Variable | Description |
| -------- | ----------- |
| `aap_red_hat_username` | This is your Red Hat account name that will be used for Subscription Management. |
| `aap_red_hat_password` | The Red Hat account password. |
| `infrastructure_database_server_user` | Username that will be the admin of the new database server. |
| `infrastructure_database_server_password` | Password of the admin of the new database server. |
| `aap_admin_password` | The admin password to create for Ansible Automation Platform application. |                      

Assuming that all variables are configured properly and your Azure account has permissions to deploy the resources defined in this collection, then running the playbook should be a single task. For example:

```bash
ansible-playbook lab.azure_deployment.deploy_infrastructure --extra-vars "aap_red_hat_username=$RED_HAT_ACCOUNT aap_red_hat_password=$RED_HAT_PASSWORD infrastructure_database_server_user=example_user infrastructure_database_server_password=example_password"
```

## Uninstall

The `destroy_infrastructure` playbook will remove RHEL subscription entitlements and deprovision the infrastructure that has been deployed in the given resource group.
This will permanently remove all data and infrastructure in the resource group, so only run this playbook if you are sure that you want to delete all traces of the deployment.

The variables below are required for running the playbook

| Variable | Description |
| -------- | ----------- |
| `aap_red_hat_username` | This is your Red Hat account name that will be used for Subscription Management. |
| `aap_red_hat_password` | The Red Hat account password. |

```bash
ansible-playbook lab.azure_deployment.destroy_infrastructure --extra-vars "aap_red_hat_username=$RED_HAT_ACCOUNT aap_red_hat_password=$RED_HAT_PASSWORD"
```
