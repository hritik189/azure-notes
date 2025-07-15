# Terraform

## Introduction to Terraform

>Terraform is an **Infrastructure as Code (IaC)** tool developed by HashiCorp that enables you to provision, automate, and manage your infrastructure, platform, and services. Terraform uses a declarative configuration language called HashiCorp Configuration Language (HCL) instead of procedural programming languages. When you invoke Terraform on a resource, it performs CRUD (Create, Read, Update, Delete) actions on your behalf to achieve the desired state of your infrastructure.

### Key Highlights:
- **Plugin-based architecture**: Extensible via providers and provisioners
- **Declarative approach**: Define *what* infrastructure should look like, not *how* to build it
- **Immutable infrastructure**: Promotes safe and predictable deployments
- **Multi-cloud support**: Works with Azure, AWS, GCP, and more
- **Version control**: Infrastructure changes can be tracked and rolled back
- **State management**: Maintains a state file to track resource changes
- **Plan before apply**: Preview changes before implementation

---

## Infrastructure as Code (IaC)

>The intention behind Infrastructure as Code (IaC) is to define, create, change, and destroy the intended infrastructure by writing and running code. One of the important ideas of DevOps is that you can manage almost everything by writing code, including systems, databases, servers, networks, and more.

### Categories of Infrastructure as Code (IaC) Tools

`Infrastructure as Code tools can be broadly divided into **five major categories**:`

#### 1. Ad Hoc Scripts
- **Description**: The simplest form of automation
- **How it works**: Manual tasks are broken into discrete steps and scripted using languages like **Bash** or **PowerShell**
- **Use Case**: Quick automation for small-scale or one-off tasks

#### 2. Configuration Management Tools
- **Purpose**: Install and manage software on existing systems
- **Popular Tools**: Chef, Puppet, Ansible
- **Use Case**: Ensuring consistent configuration across multiple servers

#### 3. Server Templating Tools
- **Purpose**: Create pre-configured server images
- **How it works**: Build a fully configured image once, then replicate it
- **Popular Tools**: Docker, Packer, Vagrant
- **Use Case**: Rapid deployment of identical environments

#### 4. Orchestration Tools
- **Purpose**: Manage and coordinate containers or services
- **Popular Tools**: Kubernetes, Azure Container Instances, Docker Swarm
- **Use Case**: Scaling, networking, and managing containerized applications

#### 5. Provisioning Tools
- **Purpose**: Create infrastructure resources like servers, databases, and networks
- **Popular Tools**: Terraform, Azure Resource Manager (ARM), Bicep
- **Use Case**: Define and deploy infrastructure in a repeatable and controlled manner

### Benefits of Infrastructure as Code (IaC)

#### 1. **Speed and Efficiency**
- Automates infrastructure provisioning, reducing setup time from hours to minutes
- Enables rapid scaling and deployment of environments

#### 2. **Consistency and Standardization**
- Eliminates configuration drift by ensuring environments are identical
- Reduces human error through repeatable, version-controlled scripts

#### 3. **Version Control and Auditability**
- Infrastructure definitions stored in source control (Git)
- Enables rollbacks, change tracking, and collaboration

#### 4. **Cost Management**
- Easily tear down unused environments to avoid unnecessary costs
- Automate cost-effective infrastructure provisioning

#### 5. **Improved Collaboration**
- Teams can work together using the same codebase
- Encourages DevOps practices and infrastructure ownership

#### 6. **Disaster Recovery and Resilience**
- Infrastructure can be quickly rebuilt from code
- Facilitates backup and recovery strategies

#### 7. **Testing and Validation**
- Infrastructure can be tested like application code
- Enables CI/CD for infrastructure

#### 8. **Security and Compliance**
- Enforces security policies through code
- Enables automated compliance checks

---

## Use Cases of Terraform

### 1. **Multi-Cloud Deployment**
- Deploy resources across Azure, AWS, and GCP simultaneously
- Avoid vendor lock-in by using consistent tooling

### 2. **Environment Provisioning**
- Create identical development, staging, and production environments
- Ensure consistency across all environments

### 3. **Disaster Recovery**
- Quickly rebuild infrastructure in different regions
- Automate backup infrastructure provisioning

### 4. **Application Deployment**
- Provision underlying infrastructure for applications
- Integrate with CI/CD pipelines for automated deployments

### 5. **Network Infrastructure**
- Create complex network topologies
- Manage VNets, subnets, NSGs, and load balancers

### 6. **Database Management**
- Provision and configure databases
- Manage database replicas and backups

### 7. **Monitoring and Logging**
- Set up monitoring solutions like Azure Monitor
- Configure log analytics workspaces

---

## How Terraform Works

### Terraform Workflow

1. **Write Configuration**: Define infrastructure in HCL files
2. **Initialize**: Run `terraform init` to download providers
3. **Plan**: Run `terraform plan` to preview changes
4. **Apply**: Run `terraform apply` to create/modify infrastructure
5. **Manage**: Use state file to track and manage resources

### Terraform Core Components

#### 1. **Terraform Core**
- Main engine that reads configuration files
- Manages state and creates execution plans
- Communicates with providers

#### 2. **Providers**
- Plugins that interact with APIs of cloud platforms
- Translate Terraform configurations to API calls
- Examples: AzureRM, AWS, Google Cloud

#### 3. **Resources**
- Fundamental building blocks of Terraform
- Represent infrastructure objects
- Examples: Virtual machines, storage accounts, networks

#### 4. **Data Sources**
- Read-only information from existing infrastructure
- Used to reference resources not managed by Terraform
- Examples: Existing resource groups, images, DNS zones

---

## Components of Terraform

### 1. **Configuration Files (.tf)**
- Written in HashiCorp Configuration Language (HCL)
- Define desired infrastructure state
- Typically organized into multiple files

### 2. **State File (terraform.tfstate)**
- JSON file that tracks current infrastructure state
- Maps configuration to real-world resources
- Should be stored securely and shared for team collaboration

### 3. **Providers**
- Plugins that enable Terraform to interact with APIs
- Must be declared and configured in your configuration

### 4. **Resources and Data Sources**
- Resources create and manage infrastructure
- Data sources read information from existing infrastructure

### 5. **Variables and Outputs**
- Variables make configurations reusable
- Outputs extract information from resources

### 6. **Backend Configuration**
- Determines where state is stored
- Enables team collaboration and state locking

---

## Terraform Modules

>A Terraform module is a container for multiple resources that are used together. Every Terraform configuration has at least one module—the root module. You can also create child modules to encapsulate and reuse code.

### Why Use Modules?
- **Reusability**: Write once, use in multiple places
- **Organization**: Break complex infrastructure into manageable parts
- **Consistency**: Standardize infrastructure patterns
- **Collaboration**: Share modules across teams or projects
- **Maintainability**: Easier to update and maintain infrastructure

### Module Structure
```
azure-resource-group/
├── main.tf       # Resource definitions
├── variables.tf  # Input variables
├── outputs.tf    # Output values
├── README.md     # Documentation
└── versions.tf   # Provider version constraints
```

### Example Module Implementation

**main.tf**
```hcl
resource "azurerm_resource_group" "this" {
  name     = var.name
  location = var.location
  tags     = var.tags
}

resource "azurerm_storage_account" "this" {
  count                    = var.create_storage_account ? 1 : 0
  name                     = var.storage_account_name
  resource_group_name      = azurerm_resource_group.this.name
  location                 = azurerm_resource_group.this.location
  account_tier             = var.storage_account_tier
  account_replication_type = var.storage_account_replication_type
  tags                     = var.tags
}
```

**variables.tf**
```hcl
variable "name" {
  description = "Name of the resource group"
  type        = string
}

variable "location" {
  description = "Azure region where resources will be created"
  type        = string
}

variable "tags" {
  description = "Tags to be applied to resources"
  type        = map(string)
  default     = {}
}

variable "create_storage_account" {
  description = "Whether to create a storage account"
  type        = bool
  default     = false
}

variable "storage_account_name" {
  description = "Name of the storage account"
  type        = string
  default     = ""
}

variable "storage_account_tier" {
  description = "Storage account tier"
  type        = string
  default     = "Standard"
}

variable "storage_account_replication_type" {
  description = "Storage account replication type"
  type        = string
  default     = "LRS"
}
```

**outputs.tf**
```hcl
output "resource_group_name" {
  description = "Name of the created resource group"
  value       = azurerm_resource_group.this.name
}

output "resource_group_id" {
  description = "ID of the created resource group"
  value       = azurerm_resource_group.this.id
}

output "location" {
  description = "Location of the resource group"
  value       = azurerm_resource_group.this.location
}

output "storage_account_id" {
  description = "ID of the storage account"
  value       = var.create_storage_account ? azurerm_storage_account.this[0].id : null
}
```

**versions.tf**
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}
```

### Root Module Usage

```hcl
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

module "dev_environment" {
  source = "./azure-resource-group"

  name                         = "rg-dev-eastus"
  location                     = "East US"
  create_storage_account       = true
  storage_account_name         = "stdeveastus001"
  storage_account_tier         = "Standard"
  storage_account_replication_type = "LRS"

  tags = {
    Environment = "Development"
    Project     = "MyApp"
    Owner       = "DevTeam"
  }
}

module "prod_environment" {
  source = "./azure-resource-group"

  name                         = "rg-prod-eastus"
  location                     = "East US"
  create_storage_account       = true
  storage_account_name         = "stprodeastus001"
  storage_account_tier         = "Premium"
  storage_account_replication_type = "GRS"

  tags = {
    Environment = "Production"
    Project     = "MyApp"
    Owner       = "OpsTeam"
  }
}

output "dev_resource_group_name" {
  value = module.dev_environment.resource_group_name
}

output "prod_resource_group_name" {
  value = module.prod_environment.resource_group_name
}
```

---

## Terraform Resource Configuration

>A resource in Terraform represents a piece of infrastructure, such as a virtual network, compute instance, or storage account. Resources are the most important element in the Terraform language.

### Basic Structure of a Resource Block

```hcl
resource "<PROVIDER>_<TYPE>" "<NAME>" {
  # Configuration arguments
}
```

- **`<PROVIDER>`**: The cloud provider (e.g., `azurerm`)
- **`<TYPE>`**: The type of resource (e.g., `resource_group`, `virtual_machine`)
- **`<NAME>`**: A local name used to reference this resource

### Common Azure Resource Examples

#### Resource Group
```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-terraform-demo"
  location = "East US"

  tags = {
    Environment = "Development"
    Project     = "TerraformDemo"
  }
}
```

#### Virtual Network
```hcl
resource "azurerm_virtual_network" "main" {
  name                = "vnet-terraform-demo"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  tags = {
    Environment = "Development"
  }
}
```

#### Subnet
```hcl
resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}
```

#### Network Security Group
```hcl
resource "azurerm_network_security_group" "main" {
  name                = "nsg-terraform-demo"
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
```

#### Virtual Machine
```hcl
resource "azurerm_linux_virtual_machine" "main" {
  name                = "vm-terraform-demo"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "adminuser"

  disable_password_authentication = true

  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]

  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts-gen2"
    version   = "latest"
  }
}
```

### Resource Meta-Arguments

#### count
```hcl
resource "azurerm_virtual_machine" "web" {
  count = 3
  name  = "web-vm-${count.index}"
  # ... other configuration
}
```

#### for_each
```hcl
resource "azurerm_resource_group" "environments" {
  for_each = toset(["dev", "staging", "prod"])
  name     = "rg-${each.key}"
  location = "East US"
}
```

#### depends_on
```hcl
resource "azurerm_virtual_machine" "web" {
  # ... configuration
  depends_on = [azurerm_network_security_group.main]
}
```

#### lifecycle
```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-important"
  location = "East US"

  lifecycle {
    prevent_destroy = true
    ignore_changes  = [tags]
  }
}
```

---

## Variables and Outputs

### Variables

`Variables allow you to parameterize your Terraform configurations, making them reusable and flexible.`

#### Variable Types

```hcl
# String variable
variable "resource_group_name" {
  description = "Name of the resource group"
  type        = string
  default     = "rg-default"
}

# Number variable
variable "vm_count" {
  description = "Number of VMs to create"
  type        = number
  default     = 1
}

# Boolean variable
variable "enable_monitoring" {
  description = "Enable monitoring for resources"
  type        = bool
  default     = true
}

# List variable
variable "allowed_ips" {
  description = "List of allowed IP addresses"
  type        = list(string)
  default     = ["10.0.0.0/8", "192.168.0.0/16"]
}

# Map variable
variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default = {
    Environment = "Development"
    Project     = "MyApp"
  }
}

# Object variable
variable "vm_config" {
  description = "Virtual machine configuration"
  type = object({
    name = string
    size = string
    os   = string
  })
  default = {
    name = "default-vm"
    size = "Standard_B1s"
    os   = "Ubuntu"
  }
}
```

#### Variable Validation

```hcl
variable "location" {
  description = "Azure region"
  type        = string

  validation {
    condition     = contains(["East US", "West US", "Central US"], var.location)
    error_message = "Location must be one of: East US, West US, Central US."
  }
}

variable "vm_size" {
  description = "Size of the virtual machine"
  type        = string

  validation {
    condition     = can(regex("^Standard_", var.vm_size))
    error_message = "VM size must start with 'Standard_'."
  }
}
```

#### Using Variables

```hcl
resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location
  tags     = var.tags
}

resource "azurerm_virtual_machine" "web" {
  count = var.vm_count
  name  = "${var.vm_config.name}-${count.index}"
  size  = var.vm_config.size
  # ... other configuration
}
```

#### Variable Files

**terraform.tfvars**
```hcl
resource_group_name = "rg-production"
location           = "East US"
vm_count           = 3
enable_monitoring  = true

tags = {
  Environment = "Production"
  Project     = "WebApp"
  Owner       = "DevOps Team"
}

vm_config = {
  name = "web-server"
  size = "Standard_D2s_v3"
  os   = "Ubuntu"
}
```

**dev.tfvars**
```hcl
resource_group_name = "rg-development"
location           = "Central US"
vm_count           = 1
enable_monitoring  = false

tags = {
  Environment = "Development"
  Project     = "WebApp"
  Owner       = "Dev Team"
}
```

### Outputs

`Outputs allow you to extract and display information from your Terraform resources after deployment.`

#### Basic Outputs

```hcl
output "resource_group_name" {
  description = "Name of the resource group"
  value       = azurerm_resource_group.main.name
}

output "resource_group_id" {
  description = "ID of the resource group"
  value       = azurerm_resource_group.main.id
}

output "virtual_network_id" {
  description = "ID of the virtual network"
  value       = azurerm_virtual_network.main.id
}

output "subnet_ids" {
  description = "IDs of the subnets"
  value       = azurerm_subnet.internal[*].id
}
```

#### Sensitive Outputs

```hcl
output "admin_password" {
  description = "Admin password for the virtual machine"
  value       = azurerm_linux_virtual_machine.main.admin_password
  sensitive   = true
}
```

#### Complex Outputs

```hcl
output "vm_details" {
  description = "Details of created virtual machines"
  value = {
    for vm in azurerm_linux_virtual_machine.web :
    vm.name => {
      id                = vm.id
      private_ip        = vm.private_ip_address
      public_ip         = vm.public_ip_address
      resource_group    = vm.resource_group_name
    }
  }
}
```

---

## State Management

>Terraform uses a state file to keep track of the infrastructure it manages. This file is a snapshot of your infrastructure stored in `terraform.tfstate` and tracks metadata like resource IDs, dependencies, and attributes.

### Why is State Important?

- **Mapping**: Connects Terraform configuration to real infrastructure
- **Performance**: Speeds up operations by caching resource information
- **Dependency Tracking**: Helps Terraform understand resource relationships
- **Change Detection**: Identifies what needs to be created, updated, or destroyed
- **Collaboration**: Enables multiple team members to work on the same infrastructure

### State Storage Types

| Type       | Description                                                                                   |
|------------|-----------------------------------------------------------------------------------------------|
| **Local**  | Default. Stored on your local machine. Good for testing and individual development.           |
| **Remote** | Stored in a shared backend (e.g., Azure Storage, S3). Enables collaboration and state locking. |

### Remote State Configuration

#### Azure Storage Backend

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate001"
    container_name       = "tfstate"
    key                  = "dev/terraform.tfstate"
  }
}
```

#### Creating Azure Storage for State

```hcl
# Create resource group for state storage
resource "azurerm_resource_group" "state" {
  name     = "rg-terraform-state"
  location = "East US"
}

# Create storage account for state
resource "azurerm_storage_account" "state" {
  name                     = "stterraformstate001"
  resource_group_name      = azurerm_resource_group.state.name
  location                 = azurerm_resource_group.state.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  tags = {
    Purpose = "Terraform State Storage"
  }
}

# Create container for state files
resource "azurerm_storage_container" "state" {
  name                  = "tfstate"
  storage_account_name  = azurerm_storage_account.state.name
  container_access_type = "private"
}
```

### State Commands

#### List Resources
```bash
terraform state list
```

#### Show Resource Details
```bash
terraform state show azurerm_resource_group.main
```

#### Remove Resource from State
```bash
terraform state rm azurerm_resource_group.main
```

#### Move/Rename Resource
```bash
terraform state mv azurerm_resource_group.old azurerm_resource_group.new
```

#### Import Existing Resource
```bash
terraform import azurerm_resource_group.main /subscriptions/subscription-id/resourceGroups/existing-rg
```

#### Refresh State
```bash
terraform refresh
```

### State Locking

State locking prevents multiple users from modifying the same state simultaneously.

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate001"
    container_name       = "tfstate"
    key                  = "dev/terraform.tfstate"
    use_azuread_auth     = true
  }
}
```

### Best Practices for State Management

1. **Always use remote state** for team collaboration
2. **Enable state locking** to prevent conflicts
3. **Use separate state files** for different environments
4. **Backup state files** regularly
5. **Restrict access** to state storage
6. **Never edit state files** manually
7. **Use workspaces** for environment isolation

---

## Terraform CLI Commands

### Initialization
```bash
terraform init                    # Initialize configuration directory
terraform init -upgrade           # Upgrade providers to latest versions
terraform init -reconfigure       # Reconfigure backend
```

### Validation & Formatting
```bash
terraform validate                # Validate configuration files
terraform fmt                     # Format files to canonical style
terraform fmt -recursive          # Format all files recursively
terraform fmt -check              # Check if files are formatted
```

### Planning & Applying
```bash
terraform plan                    # Show execution plan
terraform plan -out=plan.out      # Save plan to file
terraform apply                   # Apply changes
terraform apply plan.out          # Apply saved plan
terraform apply -auto-approve     # Apply without confirmation
terraform destroy                 # Destroy all resources
terraform destroy -auto-approve   # Destroy without confirmation
```

### Variables & Outputs
```bash
terraform output                  # Show all outputs
terraform output resource_group_name  # Show specific output
terraform apply -var="name=value" # Pass variable via command line
terraform apply -var-file="dev.tfvars"  # Use variable file
```

### State Management
```bash
terraform state list              # List all resources
terraform state show <resource>   # Show resource details
terraform state rm <resource>     # Remove resource from state
terraform state mv <src> <dst>    # Move/rename resource
terraform import <resource> <id>  # Import existing resource
terraform refresh                 # Refresh state
```

### Workspace Management
```bash
terraform workspace list          # List workspaces
terraform workspace new <name>    # Create new workspace
terraform workspace select <name> # Switch to workspace
terraform workspace delete <name> # Delete workspace
```

### Advanced Commands
```bash
terraform graph                   # Generate dependency graph
terraform show                    # Show current state
terraform console                 # Interactive console
terraform force-unlock <lock-id>  # Force unlock state
terraform taint <resource>        # Mark resource for recreation
terraform untaint <resource>      # Remove taint from resource
```

---

## Best Practices

### 1. **Project Structure**
```
project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
├── modules/
│   ├── azure-vm/
│   └── azure-network/
└── README.md
```

### 2. **Naming Conventions**
- Use consistent naming patterns
- Include environment in resource names
- Use descriptive names for resources and variables

### 3. **Security**
- Store sensitive data in Azure Key Vault
- Use managed identities when possible
- Implement least-privilege access
- Never commit secrets to version control

### 4. **Version Control**
- Use Git for infrastructure code
- Implement branching strategies
- Use pull requests for code review
- Tag releases for environment deployments

### 5. **CI/CD Integration**
- Automate terraform validate and plan
- Use separate pipelines for different environments
- Implement approval processes for production
- Store state in secure, centralized location

>This comprehensive guide covers all essential Terraform concepts with practical Azure examples. Each section builds upon the previous one, providing a complete learning path from basic concepts to advanced state management and best practices.
