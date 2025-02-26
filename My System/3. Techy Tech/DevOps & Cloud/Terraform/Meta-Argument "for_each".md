The `for_each` meta-argument accepts a map or a set of strings, and creates an instance for each item in that map or set we need [[toset, tomap]] for it:
```hcl
resource "aws_instance" "myec2vm" {
	for_each = toset(data.aws_availability_zones.my_azones.names)
	availability_zone = each.key
	tags = {
		"Name" = "for_each-${each.value}"
	}
}
```

In case we are using a set of strings, `each.key` and `each.value` will be the same.

[[Terraform]]