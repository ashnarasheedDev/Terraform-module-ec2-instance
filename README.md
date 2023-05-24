# Terraform-module-ec2-instance

Creating a Terraform module for resources allows you to encapsulate the configuration and reuse it across different projects or environments. Here's an example of how you can create a Terraform module for an EC2 instance

> <b><i>Step by step method</i></b>


**1. Create a new directory for your module, such as ec2-instance.**

**2. Inside the ec2-instance directory, create a file named main.tf. This file will contain the main configuration for your EC2 instance module.**

**3. Define the input variables for your module in a file named variables.tf.**


> <b>Create variables.tf</b>

```
variable "project_name" {
  type        = string
}

variable "project_env" {
  type        = string
}

variable "instance_type" {
  type        = string
}

variable "ami_id" {
  type        = string
  
}
```

> <b>Create outputs.tf</b> 


This is to define the outputs that will be exposed by your module.

```
output "instance_id" {
  value       = aws_instance.webserver.id
  description = "The ID of the created EC2 instance"
}

output "instance_public_ip" {
  value       = aws_instance.webserver.public_ip
  description = "The public IP address of the created EC2 instance"
}
```


> <b>Create main.tf</b>

 
In the main.tf file, define the EC2 instance resource block using the input variables defined in variables.tf.

```
resource "aws_instance" "webserver" {
  ami           = var.ami_id
  instance_type = var.instance_type
  tags = {
    Name = var.project_name
    Env = var.project_env
  }

  # Add other instance configurations
}
```

Add any additional configurations required for the EC2 instance, such as security groups, key pairs, or networking settings.


**In the Terraform project directory,follow these steps**


> <b> Create provider.tf </b>


```

provider "aws" {

region = "ap-south-1"

}
```

> <b>Create variables.tf to pass values</b>


```
variable "my_project" {
default = "<project-name>"
}

variable "my_env" {
default = "dev"
}

variable "ami_id" {
default = "<your-ami-id>"
}

variable "instance_type" {
default = "t2.micro"
}
```


> <b>Create a file named main.tf to use your EC2 instance module.</b>


```
module "ec2_instance" {
  source       = "./ec2-instance"  # Path to your module directory
  project_name = var.my_project
  project_env = var.my_env
  ami_id = var.ami_id
  instance_type = var.instance_type

}

```

You can now use the module by running **terraform init** and then **terraform apply** in the root directory of your main project.


By creating a Terraform module for an EC2 instance, you can reuse and maintain consistent configurations across multiple deployments, simplifying your infrastructure management.
