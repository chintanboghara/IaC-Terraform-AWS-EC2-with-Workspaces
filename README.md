# Terraform AWS EC2 with Workspaces

This repository demonstrates how to manage AWS EC2 instances using Terraform with support for multiple environments (e.g., `dev`, `stage`, and `prod`) via Terraform workspaces.

## Repository Structure

```
.
|-- modules/
|   |-- ec2_instance/
|       |-- main.tf
|
|-- main.tf
|-- terraform.tfvars
|-- README.md
```

### 1. **`modules/ec2_instance/main.tf`**

This module defines the EC2 instance resource. It includes variables for specifying the AMI ID and the instance type.

#### Code Breakdown:

```hcl
provider "aws" {
  region = "ap-south-1"
}

variable "ami" {
  description = "This is AMI for the instance"
}

variable "instance_type" {
  description = "This is the instance type, for example: t2.micro"
}

resource "aws_instance" "example" {
  ami           = var.ami
  instance_type = var.instance_type
}
```

- **`provider "aws"`**: Specifies the AWS provider and the region.
- **Variables**:
  - `ami`: Specifies the Amazon Machine Image (AMI) ID.
  - `instance_type`: Specifies the EC2 instance type.
- **Resource**:
  - `aws_instance`: Creates the EC2 instance using the specified AMI and instance type.

### 2. **`main.tf`**

The root `main.tf` integrates the `ec2_instance` module and uses Terraform workspaces to dynamically select the instance type for different environments.

#### Code Breakdown:

```hcl
provider "aws" {
  region = "ap-south-1"
}

variable "ami" {
  description = "value"
}

variable "instance_type" {
  description = "value"
  type        = map(string)

  default = {
    "dev"   = "t2.micro"
    "stage" = "t2.small"
    "prod"  = "t2.nano"
  }
}

module "ec2_instance" {
  source        = "./modules/ec2_instance"
  ami           = var.ami
  instance_type = lookup(var.instance_type, terraform.workspace, "t2.micro")
}
```

- **`provider "aws"`**: Specifies the AWS provider and the region.
- **`variable "ami"`**: Specifies the AMI ID (defined in `terraform.tfvars`).
- **`variable "instance_type"`**: Defines a map of instance types for different environments.
  - `dev`: Default instance type is `t2.micro`.
  - `stage`: Default instance type is `t2.small`.
  - `prod`: Default instance type is `t2.nano`.
- **Module Integration**:
  - Uses the `ec2_instance` module to provision EC2 instances.
  - `lookup`: Selects the instance type based on the active workspace.

### 3. **`terraform.tfvars`**

The `terraform.tfvars` file specifies values for variables.

```hcl
ami = "ami-053b12d3152c0cc71"
```

- **`ami`**: The AMI ID to use for all instances.

## How to Use This Repository

### Prerequisites

- Terraform installed on your system.
- AWS CLI configured with appropriate permissions.

### Steps to Provision Resources

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/<your-username>/terraform-aws-ec2-workspaces.git
   cd terraform-aws-ec2-workspaces
   ```

2. **Initialize Terraform**:
   ```bash
   terraform init
   ```

3. **Create a Workspace**:
   - To create and switch to a workspace:
     ```bash
     terraform workspace new <workspace-name>
     ```
     Replace `<workspace-name>` with `dev`, `stage`, or `prod`.

   - To list available workspaces:
     ```bash
     terraform workspace list
     ```

4. **Apply the Terraform Configuration**:
   ```bash
   terraform apply
   ```
   Confirm the plan to provision resources.

5. **Verify the Created Resources**:
   - Log in to your AWS Management Console.
   - Navigate to EC2 Instances to see the newly created instance.

6. **Clean Up Resources**:
   To destroy the resources:
   ```bash
   terraform destroy
   ```

## Key Features

- **Environment-Specific Configurations**: Use Terraform workspaces to manage configurations for `dev`, `stage`, and `prod` environments.
- **Modular Design**: The EC2 instance logic is encapsulated in a reusable module.
- **Dynamic Instance Types**: Selects the instance type dynamically based on the active workspace.

## Example Workspaces

| Workspace | Instance Type |
|-----------|---------------|
| dev       | t2.micro      |
| stage     | t2.small      |
| prod      | t2.nano       |
