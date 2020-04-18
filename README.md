# Rancher infra AWS terraform module 

Terraform module for deploying nodes for k8s cluster on AWS ec2. 

Module will do following tasks:
- Create keypair
- Configure sg
- Deploy configured nodes
- Deploy NLB if enabled
- Add route53 record if enabled

## Variables

### Input

This module accept the following variables as input:

```
# Required variables
variable "aws_access_key" {
  type        = string
  description = "AWS access key used to create infrastructure"
}

variable "aws_secret_key" {
  type        = string
  description = "AWS secret key used to create AWS infrastructure"
}

# Optional variables
variable "aws_region" {
  description = "AWS region used for all resources"
  default     = "us-east-1"
}

variable "route53_zone" {
  description = "AWS route53 zone"
  default     = ""
}

variable "route53_name" {
  description = "AWS route53 domain name"
  default     = "rancher"
}

variable "deploy_lb" {
  description = "Deploy AWS nlb in front of worker nodes"
  default     = false
}

variable "prefix" {
  description = "Prefix added to names of all resources"
  default     = "rancher-infra-aws"
}

variable "node_master_count" {
  description = "Master nodes count"
  default     = 0
}

variable "node_worker_count" {
  description = "Worker nodes count"
  default     = 0
}

variable "node_all_count" {
  description = "All roles nodes count"
  default     = 1
}

variable "node_username" {
  description = "Instance type used for all EC2 instances"
  default     = "ubuntu"
}

variable "instance_type" {
  description = "Instance type used for all EC2 instances"
  default     = "t3a.medium"
}

variable "docker_version" {
  description = "Docker version to install on nodes"
  default     = "19.03"
}

variable "ssh_key_file" {
  description = "File path and name of SSH private key used for infrastructure"
  default     = "~/.ssh/id_rsa"
}

variable "ssh_pub_file" {
  description = "File path and name of SSH public key used for infrastructure"
  default     = ""
}

variable "register_command" {
  description = "Register command for nodes"
  default     = ""
}

variable "user_data" {
  default = ""
}
```

### Output

This module use the following variables as ouput:

```
output "rancher_nodes" {
  value = [
  	for instance in flatten([[aws_instance.node_all], [aws_instance.node_master], [aws_instance.node_worker]]): {
    
    public_ip  = instance.public_ip
    private_ip = instance.private_ip
    hostname   = instance.id
    user       = var.node_username
    roles      = split(",", instance.tags.K8sRoles)
    ssh_key    = file(var.ssh_key_file)
    }
  ]
  sensitive = true
}
```

## How to use

This tf module can be used standalone or combined with other tf modules.

Requirements for use standalone:
* AWS credentials

Add the following to your tf file:

```
module "rancher_infra" {
  source = "github.com/rawmind0/tf-module-rancher-infra-aws"

  aws_access_key = "XXXXXXXXXXXX"
  aws_secret_key = "XXXXXXXXXXXXXXXXXXXXXXXX"
  aws_region = "eu-west-3"
  prefix = "rancher-ha"
  # deploy_lb = true  # Deploy NLB pointing to nodes with worker role
  # route53_zone = "my.org"  # Use route53 zone to add registry
  # route53_name = "rancher-ha"        
}
```


