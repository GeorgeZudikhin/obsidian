For Loop with List
```hcl
output "for_output_list" {
	description = "For Loop with List"
	value = [for instance in aws_instance.myec2vm: instance.public_dns]
}
```

For Loop with Map. Outputting a mapping of id to public_dns for each instance
```hcl
output "for_output_map" {
	description = "For Loop with Map"
	value = {for instance in aws_instance.myec2vm: instance.id => instance.public_dns}
}
```

For Loop with Advanced Map
```hcl
output "for_output_advanced_map" {
	description = "For Loop with Advanced Map"
	value = {for count, instance in aws_instance.myec2vm: count => public_dns}
}
```

Splat operator, returns a List
```hcl
output "splat_instances_publicdns" {
	description = "Generalized latest splat operator"
	value = aws_instance.myec2vm[*].public_dns
}
```

[[Terraform]]