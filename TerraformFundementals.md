

# Why state matters
## Resource tracking!
• A way for Terraform to keep tabs on what has been deployed.
• Critical to Terraforms functionality.
• Stored in flat files, by default named “terraform.tfstate”.
• Helps Terraform calculate deployment deltas and create new deployment plans.
• Never lose your Terraform state file.

# Terraform Variables and Outputs

### variable
```tf
variable “my-var” {
	description = “My test variable”
	type = string
	default = “Hello”
}
```
variable = Reserved Keyword
my-var = User provided variable name
{} = variable config arguments such as type of variable and default value.

Referencing a variable: var.my-var

Best practice is combine all the variables into “terraform.tfvars”.

### Validation

```tf
variable “my-var” {
	description = “My test variable”
	type = string
	default = “Hello”
	validation {
		condition = length(var.my-var) > 4
		error_message = “The string must be more than 4 characters”
	}
}
```
the validation above ensures the string variable is more than 4 characters.

Hidden variables
in order to hide sensitive values during the execution of the terraform logs.
```tf
variable “my-var” {
	description = “My test variable”
	type = string
	default = “Hello”
	sensitive = true
}
```

### Base types for terraform variables
• string
• number
• bool

### Complex types for terraform variables
• list
• set
• map
• object
• tuple


#### String example
```hcl
variable “image_id” {
	type = string
	default = “hello”
}
```

#### List example
```hcl
variable “availability_zone_names” {
	type = list(string)
	default = [“us-west-1a”]
}
```

#### List of Objects example
```hcl
variable “docker_ports” {
	type = list(object({
		internal = number
		external = number
		protocol = string
	}))
	default = [
		{
			internal = 8300
			external = 8300
			protocol = “tcp”
		}
	]
}
```


# Terraform Outputs

output variables values are shown on the shell after running terraform apply.
```hcl
output “instance_ip” {
	description = “VM’s private IP”
	value = aws_instance.my-vm.private_ip
}
```
Output values are like return values that you want to tract after a successful Terraform deployment

# Terraform Provisioners

- Terraforms way of bootstrapping custom scripts, commands or actions.
- Can be either locally (on the same system where Terraform commands are being issued from), or remotely on resources spun up through the Terraform deployment.
- Within Terraform code, each individual resource can have its own "provisioner" defining the connection method (if required such as SSH or WinRM) and the actions/commands or script to execute.
- There are two types of provisioners: "Creation-time" and "Destroy-time" provisioners which you can set to run when a resource is being created or destroyed.

> DISCLAIMER: Hashicorp recommends using Provisioners as a last resort and to try using inherent mechanisms within your infrastructure deployment to carry out custom tasks where possible.

Terraform cannot track changes to provisioners as they can take any independent action, hence they are not tracked by Terraform state files.

Provisioners are recommended for use when you want to invoke actions not covered by Terraforms declarative model.
If the command within a provisioner returns non-zero return code, its considered failed and underlying resource is tainted.

```hcl
resourse "null_resource" "dummy_resource" {
    provisioner "local-exec" {
        command = "echo '0' > status.txt"
    }

    provisioner "local-exec" {
        when = destory
        command = "echo '1' > status.txt"
    }
}
```

> If a "Creation-time" provisioner fails, Terraform will mark that resource as tainted, and on the next "apply" it will try to delete and recreate the resource.

```hcl
resource "aws_instance" "ec2-virtual-machine" {
    ami                         = ami-1234
    instance_type               = t2.micro
    key_name                    = aws_key_pair.master-key.key-name
    associate_public_ip_address = true
    vpc_security_group_ids      = [aws_security_group.jenkins-sg.id]
    subnet_id                   = aws_subnet.subnet.id
    provisioner "local-exec" {
        command = "aws ec2 wait instance-status-ok --region us-east-1 --instance-ids ${self.id}"
    } 
}
```

In order to get the instance id of the resource above we would type `aws_instance.ec2-virtual-machine.id`
However attempting to use this in a provisioner can cause cyclic dependency issues. To address this hashicorp has provided the `self` object which can be used access any attribute beloging to the resource.