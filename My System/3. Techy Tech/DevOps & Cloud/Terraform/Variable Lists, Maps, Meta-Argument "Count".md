Variable declaration of type **list** and **map**:
```hcl
variable "instance_type_list" {
	description = "EC2 Instance Type"
	type = list(string)
	default = ["t3.micro", "t3.small"]
}

variable "instance_type_map" {
	description = "EC2 Instance Type"
	type = map(string)
	default = {
		"dev" = "t3.micro"
		"qa" = "t3.small"
		"prod" = "t3.large"
	}
}
```

Usage in a resource definition:
```hcl
resource "aws_instance" "myec2vm" {
	# For list, use index:
	instance_type = var.instance_type_list[0] 
	
	# For map, use key:
	instance_type = var.instance_type_map["prod"] 
	
	# Meta-argument "count", will create 5 instances of this resource:
	count = 5

	# count.index - references index of a resource created with meta-argument:
	tags = {
		"Name" = "Instance-${count.index}"
	}
}
```


[[Terraform]]