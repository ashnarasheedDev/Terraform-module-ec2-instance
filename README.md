# Terraform-module-ec2-instance

Creating a Terraform module for resources allows you to encapsulate the configuration and reuse it across different projects or environments. Here's an example of how you can create a Terraform module for an EC2 instance



### Pre-requisites:

1. IAM Role (Role needs to be attached on terraform running server)
2. knowledge about AWS services especially VPC, EC2 and IP Subnetting.
3. Terraform and its installation.



> <b><i>Step by step method</i></b>


**1. Create a new directory for your module, such as ec2-instance.**

**2. Inside the ec2-instance directory, create a file named main.tf. This file will contain the main configuration for your EC2 instance module.**

**3. Define the input variables for your module in a file named variables.tf.**


> <b>Create variables.tf</b>

```
variable "frontend_ports" {
    
  type    = list
  default = ["22","80","21","443"]
    
}

variable "my_project"{
default = "zomato"
}

variable "my_env"{
default = "test"
}

variable "region" {
default = "ap-south-1"
}

variable "instance_type" {
default = "t2.micro"
}

variable "ami_id" {
default = "ami-0c768662cc797cd75"
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

 
In the main.tf file, define resource block for EC2 instance, Key_pair,SG, EIP using the input variables defined in variables.tf. 

```
## creating keypair and downloading to local end

resource "tls_private_key" "mykey" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "mykey" {
  key_name   = "${var.my_project}-${var.my_env}" # Create "myKey" to AWS
  public_key = tls_private_key.mykey.public_key_openssh
}
resource "local_file" "mykey" {
  filename        = "${var.my_project}-${var.my_env}.pem"
  content         = tls_private_key.mykey.private_key_pem
  file_permission = "400"
}


## creating sg for instance

resource "aws_security_group" "webserver" {
        
  name_prefix        = "${var.my_project}-${var.my_env}"

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}

resource "aws_security_group_rule" "webserver-rules" {
    
  for_each          = toset(var.frontend_ports)
        
  type              = "ingress"
  from_port         = each.key
  to_port           = each.key
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  ipv6_cidr_blocks  = ["::/0"]
  security_group_id = aws_security_group.webserver.id
}

## creating ec2 instance

resource "aws_instance" "webserver" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  key_name               = aws_key_pair.mykey.key_name
  vpc_security_group_ids = [aws_security_group.webserver.id]

  tags = {
    Name    = "${var.my_project}-${var.my_env}"
    Project = "${var.my_project}"
    Env     = "${var.my_env}"
  }
}

## eip for instance

resource "aws_eip" "eip" {
  instance = aws_instance.webserver.id
  vpc      = true
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
variable "region" {
default = "ap-south-1"
}

variable "project_name" {
default = "swiggy"
}

variable "project_env" {
default = "dev"
}

```


> <b>Create a file named main.tf to use your EC2 instance module.</b>


```

module "ec2-instance" {

source = "/home/ec2-user/ec2-module"

my_project = var.project_name
my_env  = var.project_env

}


```

You can now use the module by running **terraform init** and then **terraform apply** in the root directory of your main project.


By creating a Terraform module for an EC2 instance, you can reuse and maintain consistent configurations across multiple deployments, simplifying your infrastructure management.
