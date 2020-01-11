---
layout: post
title: Recovering Terraform State
date: 2017-04-26
tags: ["terraform"]
---

Here's the scenario... you have documented the steps for creating new infrastructure using Terraform including ensuring that state files are dealt with properly (remote in AWS S3). However, at some point those directions aren't followed and you now have infrastructure that is orphaned with no state. What do you do?<!--more-->

The only answer is to get familiar with the `import` command of Terraform

## Understanding the state file

At a high level Terraform state is a mapping of the actual state of your infrastructure which was created from your configurations (which are the desired state). The state file is a custom JSON hierarchy which contains the following metadata:

* version - the protocol version of the state file
* terraform_version - the version of Terraform that wrote this state file
* serial - incremented on any operation that modifies the infrastructure
* lineage - set when the state is created
* remote - used to track the metadata required for the configured remote state
* backend - used to track the configuration for the backend in use with the state
* modules - contains all the modules in a "breadth-first order"
  * path - the import path from the root module
  * outputs - any outputs declared by the module
  * resources - a mapping of the logically name resources within that level
  * depends_on - a list of things this module relies on

Another thing to consider with regards to state and Terraform in general is that it operates using the breadth-first order (BFS) algorithm. This means that it will traverse the graph data structure level-by-level. Specifically it will start with the configurations in the current working directory which it considers the "root" module and then pull in the next level of modules and so on.

## Recovering a simple Terraform

This is a very simple configuration contained within one file `main.tf`

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-b73b63a0"
  instance_type = "t2.micro"

  root_block_device {
    delete_on_termination = "true"
  }
}
```

After applying this configuration the `terraform.tfstate` contains the following. Because the configuration is contained in the *cwd* all of the configured resources are contained in the "root" module.

```
{
    "version": 3,
    "terraform_version": "0.9.3",
    "serial": 0,
    "lineage": "84d822b4-b994-4e07-9791-ab6f6b0ce3ce",
    "modules": [
        {
            "path": [
                "root"
            ],
            "outputs": {},
            "resources": {
                "aws_instance.example": {
                    "type": "aws_instance",
                    "depends_on": [],
                    "primary": {
                        "id": "i-0e3f3db1d2c5a4520",
                        "attributes": {
                            "ami": "ami-b73b63a0",
                            "associate_public_ip_address": "true",
                            "availability_zone": "us-east-1b",
                            "disable_api_termination": "false",
                            "ebs_block_device.#": "0",
                            "ebs_optimized": "false",
                            "ephemeral_block_device.#": "0",
                            "iam_instance_profile": "",
                            "id": "i-0e3f3db1d2c5a4520",
                            "instance_state": "running",
                            "instance_type": "t2.micro",
                            "ipv6_address_count": "0",
                            "ipv6_addresses.#": "0",
                            "key_name": "",
                            "monitoring": "false",
                            "network_interface_id": "eni-f77c7f12",
                            "private_dns": "ip-172-31-15-90.ec2.internal",
                            "private_ip": "172.31.15.90",
                            "public_dns": "ec2-54-145-10-15.compute-1.amazonaws.com",
                            "public_ip": "54.145.10.15",
                            "root_block_device.#": "1",
                            "root_block_device.0.delete_on_termination": "true",
                            "root_block_device.0.iops": "0",
                            "root_block_device.0.volume_size": "8",
                            "root_block_device.0.volume_type": "standard",
                            "security_groups.#": "0",
                            "source_dest_check": "true",
                            "subnet_id": "subnet-f3d52b85",
                            "tags.%": "0",
                            "tenancy": "default",
                            "vpc_security_group_ids.#": "1",
                            "vpc_security_group_ids.324294369": "sg-a74531c1"
                        },
                        "meta": {
                            "schema_version": "1"
                        },
                        "tainted": false
                    },
                    "deposed": [],
                    "provider": ""
                }
            },
            "depends_on": []
        }
    ]
}
```

In our hypothetical we no longer have this state file and thus, if we apply this Terraform, it will create a new EC2 resource. Additionally the EC2 resource that already exists is orphaned from the Terraform that should control it.

In order to regain control of this resource we will use the `import` command to recover the metadata for this resource. Import requires a resource address and the resource ID. The resource address is a combination of the resource type and resource name which for this configuration is 'aws_instance.example'. The ID in this instance is the AWS EC2 ID 'i-0e3f3db1d2c5a4520'.

```
$ terraform import aws_instance.example i-0e3f3db1d2c5a4520
aws_instance.example: Importing from ID "i-0e3f3db1d2c5a4520"...
aws_instance.example: Import complete!
  Imported aws_instance (ID: i-0e3f3db1d2c5a4520)
aws_instance.example: Refreshing state... (ID: i-0e3f3db1d2c5a4520)

Import success! The resources imported are shown above. These are
...
```

The `import` command creates a new state file pulling in information about the specified AWS Instance ID. Once complete the only difference between the new state and old will be the lineage hash.

```
{
    "version": 3,
    "terraform_version": "0.9.3",
    "serial": 0,
    "lineage": "f02e5b73-a687-420d-b13d-b9121f8c1916",
    "modules": [
        {
            "path": [
                "root"
            ],
            "outputs": {},
            "resources": {
                "aws_instance.example": {
                    "type": "aws_instance",
                    "depends_on": [],
                    "primary": {
                        "id": "i-0e3f3db1d2c5a4520",
                        "attributes": {
                            "ami": "ami-b73b63a0",
                            "associate_public_ip_address": "true",
                            "availability_zone": "us-east-1b",
                            "disable_api_termination": "false",
                            "ebs_block_device.#": "0",
                            "ebs_optimized": "false",
                            "ephemeral_block_device.#": "0",
                            "iam_instance_profile": "",
                            "id": "i-0e3f3db1d2c5a4520",
                            "instance_state": "running",
                            "instance_type": "t2.micro",
                            "ipv6_address_count": "0",
                            "ipv6_addresses.#": "0",
                            "key_name": "",
                            "monitoring": "false",
                            "network_interface_id": "eni-f77c7f12",
                            "private_dns": "ip-172-31-15-90.ec2.internal",
                            "private_ip": "172.31.15.90",
                            "public_dns": "ec2-54-145-10-15.compute-1.amazonaws.com",
                            "public_ip": "54.145.10.15",
                            "root_block_device.#": "1",
                            "root_block_device.0.delete_on_termination": "true",
                            "root_block_device.0.iops": "0",
                            "root_block_device.0.volume_size": "8",
                            "root_block_device.0.volume_type": "standard",
                            "security_groups.#": "0",
                            "source_dest_check": "true",
                            "subnet_id": "subnet-f3d52b85",
                            "tags.%": "0",
                            "tenancy": "default",
                            "vpc_security_group_ids.#": "1",
                            "vpc_security_group_ids.324294369": "sg-a74531c1"
                        },
                        "meta": {
                            "schema_version": "1"
                        },
                        "tainted": false
                    },
                    "deposed": [],
                    "provider": "aws"
                }
            },
            "depends_on": []
        }
    ]
}
```

## Recovering a (more) complex Terraform

For purposes of this post I'm not going to get too complex, but importing modules does require some additional understanding. The Terraform that follows makes use of modules to fully define the EC2 instances (web), Security Groups (sg) and Elastic Load Balancers (elb).

```
variable "region" { default = "us-east-1" }
variable "availability_zone" { default = "us-east-1a" }
variable "instance_type" { default = "t2.micro" }
variable "instance_count" { default = "2" }
variable "key_name" {}
variable "public_key_path" {}
variable "connection_user" { default = "ec2-user" }
variable "name" { default = "web" }
variable "environment" { default = "production" }

provider "aws" {
  region = "${var.region}"
}

module "web" {
  source = "../modules/web"

  availability_zone = "${var.availability_zone}"
  connection_user = "${var.connection_user}"
  instance_count = "${var.instance_count}"
  instance_type = "${var.instance_type}"
  key_name = "${var.key_name}"
  public_key_path = "${var.public_key_path}"
  region = "${var.region}"
  web_security_groups = "${module.sg.security_group_id}"
  name = "${var.name}"
  environment = "${var.environment}"

}

module "sg" {
  source = "../modules/sg"

  name = "${var.name}"
  environment = "${var.environment}"
}

module "elb" {
  source = "../modules/elb"

  availability_zones = "${var.availability_zone}"
  name = "${var.name}"
  environment = "${var.environment}"
  web_instance_ids = "${module.web.web_instance_ids}"
}

output "elb_dns_name" {
  value = "${module.elb.dns_name}"
}
```

Running an `apply` on this Terraform yields the following state. Notice the root module now only contains output metadata. The next level are the web, sg, and elb modules referenced by the root module (recall breadth-first order). For this configuration these modules are where all of the AWS resources are created.

```
{
    "version": 3,
    "terraform_version": "0.9.3",
    "serial": 0,
    "lineage": "ae1f2572-8fa6-4977-be73-3deac7529eff",
    "modules": [
        {
            "path": [
                "root"
            ],
            "outputs": {
                "elb_dns_name": {
                    "sensitive": false,
                    "type": "string",
                    "value": "web-elb-1019323532.us-east-1.elb.amazonaws.com"
                }
            },
            "resources": {},
            "depends_on": []
        },
        {
            "path": [
                "root",
                "elb"
            ],
            "outputs": {
                "dns_name": {
                    "sensitive": false,
                    "type": "string",
                    "value": "web-elb-1019323532.us-east-1.elb.amazonaws.com"
                }
            },
            "resources": {
                "aws_elb.elb": {
                    "type": "aws_elb",
                    "depends_on": [],
                    "primary": {
                        "id": "web-elb",
                        "attributes": {
                            "access_logs.#": "0",
                            "availability_zones.#": "1",
                            "availability_zones.3569565595": "us-east-1a",
                            "connection_draining": "true",
                            "connection_draining_timeout": "400",
                            "cross_zone_load_balancing": "true",
                            "dns_name": "web-elb-1019323532.us-east-1.elb.amazonaws.com",
                            "health_check.#": "1",
                            "health_check.0.healthy_threshold": "2",
                            "health_check.0.interval": "30",
                            "health_check.0.target": "HTTP:80/",
                            "health_check.0.timeout": "5",
                            "health_check.0.unhealthy_threshold": "2",
                            "id": "web-elb",
                            "idle_timeout": "400",
                            "instances.#": "2",
                            "instances.1111961264": "i-03676fa6ba43fbb9f",
                            "instances.1714554663": "i-09f51a313146856cd",
                            "internal": "false",
                            "listener.#": "1",
                            "listener.3057123346.instance_port": "80",
                            "listener.3057123346.instance_protocol": "http",
                            "listener.3057123346.lb_port": "80",
                            "listener.3057123346.lb_protocol": "http",
                            "listener.3057123346.ssl_certificate_id": "",
                            "name": "web-elb",
                            "security_groups.#": "1",
                            "security_groups.979167962": "sg-74c46809",
                            "source_security_group": "1234567890123/default_elb_15702516-bb9f-3fbd-b08e-4ab8ab664325",
                            "source_security_group_id": "sg-74c46809",
                            "subnets.#": "1",
                            "subnets.3664814999": "subnet-690a0942",
                            "tags.%": "2",
                            "tags.Environment": "production",
                            "tags.Name": "web-elb",
                            "zone_id": "Z35SXDOTRQ7X7K"
                        },
                        "meta": {},
                        "tainted": false
                    },
                    "deposed": [],
                    "provider": ""
                }
            },
            "depends_on": []
        },
        {
            "path": [
                "root",
                "sg"
            ],
            "outputs": {
                "security_group_id": {
                    "sensitive": false,
                    "type": "string",
                    "value": "sg-5a677425"
                }
            },
            "resources": {
                "aws_security_group.web_sg": {
                    "type": "aws_security_group",
                    "depends_on": [],
                    "primary": {
                        "id": "sg-5a677425",
                        "attributes": {
                            "description": "Security group for web production",
                            "egress.#": "1",
                            "egress.482069346.cidr_blocks.#": "1",
                            "egress.482069346.cidr_blocks.0": "0.0.0.0/0",
                            "egress.482069346.from_port": "0",
                            "egress.482069346.ipv6_cidr_blocks.#": "0",
                            "egress.482069346.prefix_list_ids.#": "0",
                            "egress.482069346.protocol": "-1",
                            "egress.482069346.security_groups.#": "0",
                            "egress.482069346.self": "false",
                            "egress.482069346.to_port": "0",
                            "id": "sg-5a677425",
                            "ingress.#": "2",
                            "ingress.2214680975.cidr_blocks.#": "1",
                            "ingress.2214680975.cidr_blocks.0": "0.0.0.0/0",
                            "ingress.2214680975.from_port": "80",
                            "ingress.2214680975.ipv6_cidr_blocks.#": "0",
                            "ingress.2214680975.protocol": "tcp",
                            "ingress.2214680975.security_groups.#": "0",
                            "ingress.2214680975.self": "false",
                            "ingress.2214680975.to_port": "80",
                            "ingress.2541437006.cidr_blocks.#": "1",
                            "ingress.2541437006.cidr_blocks.0": "0.0.0.0/0",
                            "ingress.2541437006.from_port": "22",
                            "ingress.2541437006.ipv6_cidr_blocks.#": "0",
                            "ingress.2541437006.protocol": "tcp",
                            "ingress.2541437006.security_groups.#": "0",
                            "ingress.2541437006.self": "false",
                            "ingress.2541437006.to_port": "22",
                            "name": "web-production-sg",
                            "owner_id": "1234567890123",
                            "tags.%": "0",
                            "vpc_id": "vpc-16cb4b72"
                        },
                        "meta": {},
                        "tainted": false
                    },
                    "deposed": [],
                    "provider": ""
                }
            },
            "depends_on": []
        },
        {
            "path": [
                "root",
                "web"
            ],
            "outputs": {
                "web_instance_ids": {
                    "sensitive": false,
                    "type": "string",
                    "value": "i-03676fa6ba43fbb9f,i-09f51a313146856cd"
                },
                "web_public_ips": {
                    "sensitive": false,
                    "type": "string",
                    "value": "34.207.194.186,34.203.236.205"
                }
            },
            "resources": {
                "aws_instance.web.0": {
                    "type": "aws_instance",
                    "depends_on": [],
                    "primary": {
                        "id": "i-03676fa6ba43fbb9f",
                        "attributes": {
                            "ami": "ami-b73b63a0",
                            "associate_public_ip_address": "true",
                            "availability_zone": "us-east-1a",
                            "disable_api_termination": "false",
                            "ebs_block_device.#": "0",
                            "ebs_optimized": "false",
                            "ephemeral_block_device.#": "0",
                            "iam_instance_profile": "",
                            "id": "i-03676fa6ba43fbb9f",
                            "instance_state": "running",
                            "instance_type": "t2.micro",
                            "ipv6_address_count": "0",
                            "ipv6_addresses.#": "0",
                            "key_name": "aws-us-east",
                            "monitoring": "false",
                            "network_interface_id": "eni-947bbf62",
                            "private_dns": "ip-172-31-50-15.ec2.internal",
                            "private_ip": "172.31.50.15",
                            "public_dns": "ec2-34-207-194-186.compute-1.amazonaws.com",
                            "public_ip": "34.207.194.186",
                            "root_block_device.#": "1",
                            "root_block_device.0.delete_on_termination": "true",
                            "root_block_device.0.iops": "0",
                            "root_block_device.0.volume_size": "8",
                            "root_block_device.0.volume_type": "standard",
                            "security_groups.#": "0",
                            "source_dest_check": "true",
                            "subnet_id": "subnet-690a0942",
                            "tags.%": "0",
                            "tenancy": "default",
                            "vpc_security_group_ids.#": "1",
                            "vpc_security_group_ids.2555476107": "sg-5a677425"
                        },
                        "meta": {
                            "schema_version": "1"
                        },
                        "tainted": false
                    },
                    "deposed": [],
                    "provider": ""
                },
                "aws_instance.web.1": {
                    "type": "aws_instance",
                    "depends_on": [],
                    "primary": {
                        "id": "i-09f51a313146856cd",
                        "attributes": {
                            "ami": "ami-b73b63a0",
                            "associate_public_ip_address": "true",
                            "availability_zone": "us-east-1a",
                            "disable_api_termination": "false",
                            "ebs_block_device.#": "0",
                            "ebs_optimized": "false",
                            "ephemeral_block_device.#": "0",
                            "iam_instance_profile": "",
                            "id": "i-09f51a313146856cd",
                            "instance_state": "running",
                            "instance_type": "t2.micro",
                            "ipv6_address_count": "0",
                            "ipv6_addresses.#": "0",
                            "key_name": "aws-us-east",
                            "monitoring": "false",
                            "network_interface_id": "eni-0679bdf0",
                            "private_dns": "ip-172-31-63-104.ec2.internal",
                            "private_ip": "172.31.63.104",
                            "public_dns": "ec2-34-203-236-205.compute-1.amazonaws.com",
                            "public_ip": "34.203.236.205",
                            "root_block_device.#": "1",
                            "root_block_device.0.delete_on_termination": "true",
                            "root_block_device.0.iops": "0",
                            "root_block_device.0.volume_size": "8",
                            "root_block_device.0.volume_type": "standard",
                            "security_groups.#": "0",
                            "source_dest_check": "true",
                            "subnet_id": "subnet-690a0942",
                            "tags.%": "0",
                            "tenancy": "default",
                            "vpc_security_group_ids.#": "1",
                            "vpc_security_group_ids.2555476107": "sg-5a677425"
                        },
                        "meta": {
                            "schema_version": "1"
                        },
                        "tainted": false
                    },
                    "deposed": [],
                    "provider": ""
                }
            },
            "depends_on": []
        }
    ]
}
```

Again, using our hypothetical we have lost the state files for this configuration and have orphaned resources in "production". The method to recover is essentially the same except we must now include the module information in the resource address. Meaning the resource type and name needs to be prefixed with "module" and the module name. Otherwise Terraform will import the resource into the root module which will not match with the configuration.


### Web module import

Starting with the web module we must account for it having an "instance_count = 2". So we will actually need to run the import command twice to pull in information for each instance. Additionally, because we have more than one we must introduce an index to the address because we have multiple instances.

```
$ terraform import module.web.aws_instance.web[0] i-03676fa6ba43fbb9f
module.web.aws_instance.web: Importing from ID "i-03676fa6ba43fbb9f"...
module.web.aws_instance.web: Import complete!
  Imported aws_instance (ID: i-03676fa6ba43fbb9f)
module.web.aws_instance.web: Refreshing state... (ID: i-03676fa6ba43fbb9f)

Import success! The resources imported are shown above. These are
...

$ terraform import module.web.aws_instance.web[1] i-09f51a313146856cd
module.web.aws_instance.web: Importing from ID "i-09f51a313146856cd"...
module.web.aws_instance.web: Import complete!
  Imported aws_instance (ID: i-09f51a313146856cd)
module.web.aws_instance.web: Refreshing state... (ID: i-09f51a313146856cd)

Import success! The resources imported are shown above. These are
...
```

### Security group module import

Importing the security group you will notice that it creates one `aws_security_group_rule` for each rule in addition to the `aws_security_group`. This is expected behavior even though the rules in the configuration are in-line in the group. Best practice is apparently one resource per rule and I suppose I should be following best practices here, but I'm not... in the case of this configuration the extraneous rule resources will just be deleted on first `apply`.

```
$ terraform import module.sg.aws_security_group.web_sg sg-5a677425
module.sg.aws_security_group.web_sg: Importing from ID "sg-5a677425"...
module.sg.aws_security_group.web_sg: Import complete!
  Imported aws_security_group (ID: sg-5a677425)
  Imported aws_security_group_rule (ID: sgrule-3870574835)
  Imported aws_security_group_rule (ID: sgrule-1346245958)
  Imported aws_security_group_rule (ID: sgrule-1495027832)
module.sg.aws_security_group.web_sg: Refreshing state... (ID: sg-5a677425)
module.sg.aws_security_group_rule.web_sg-2: Refreshing state... (ID: sgrule-1495027832)
module.sg.aws_security_group_rule.web_sg: Refreshing state... (ID: sgrule-3870574835)
module.sg.aws_security_group_rule.web_sg-1: Refreshing state... (ID: sgrule-1346245958)

Import success! The resources imported are shown above. These are
...
```

### ELB module import

And finally we import the ELB which doesn't require anything special.

```
$ terraform import module.elb.aws_elb.elb web-elb
module.elb.aws_elb.elb: Importing from ID "web-elb"...
module.elb.aws_elb.elb: Import complete!
  Imported aws_elb (ID: web-elb)
module.elb.aws_elb.elb: Refreshing state... (ID: web-elb)

Import success! The resources imported are shown above. These are
...
```

For each of these I had to peek back into the module code to ensure that I was using the correct names for each resource in addition to pulling the proper ID from the AWS console.

The resulting state file won't be nearly as close to a match as the simple Terraform above, but it will be good enough to safely move forward with changes without fear that Terraform will destroy running stuff. That said, your infra is all immutable anyway, so if it destroys and creates new instances your application won't miss a beat. Right? Right?!
