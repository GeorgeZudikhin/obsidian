## Terraform block

**Terraform** **block** - contains settings related to terraform behavior. It can only use constant values, no variables.
Block - opens directly with *{*
Argument - opens with *= {*
```hcl
terraform {
	required_version = "~> 1.10.5"
	required_providers {
		# local provider name: module specific and should be unique per module
		aws = { 
			source = "hashicorp/aws"
			version = "~> 3.34.0"
		}
	}
}
```

## Provider block

**Provider block** - configuring the settings of a particular provider
```hcl
# name should match the local provider name specified in the terraform block
provider "aws" {
	region = "us-east-1"
}
```
## Resource block

```hcl
resource "resource_type" "resource_local_name" {
	# universal meta-argument used to change the behavior of a resource
	provider = aws.aws-west-1
	# resource argument specific to resource type
	cidr_block = "10.2.0.0/16"
}
```
**resource_local_name** is used to refer to this resource from anywhere within the module, but it cannot be referred to outside the module. 
**resource_type + resource_local_name** serve as a unique resource identifier in the module.

[[Terraform]]