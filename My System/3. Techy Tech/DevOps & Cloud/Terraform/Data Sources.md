**Data Sources** allow us to fetch information defined outside of current Terraform configuration and use it. A data source supports same meta arguments as a resource and is defined as a data. Data source types are specified like resource types and can be looked up as well.
```hcl
data "aws_ami" "amzlinux" {
	most_recent = true
	owners = ["amazon"]
	filter {
		name = "name"
		values = ["amzn2-ami-hvm-8"]
	}
}
```

[[Terraform]]