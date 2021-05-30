# IaC with Terraform

### The core of Terraform work flow

> Write -> Plan -> Apply

- Write your Terraform code.
    - This would generally start of with creating a Github repository as a common best practice.
- Review
    - You will continually add and review changes to code in your project.
- Deploy
    - After last review / plan, You will be ready to provision the infrastructure.


`terraform init`:
- Initializes the working directory that contains your Terraform code.
- Downloads Ancillary components ( modules and plugins ).
- Sets up the backend for storing Terraform state file. A mechanism by which Terraform tracks resources.

`terraform plan`:
- Reads the code, create and shows a plan of execution / deployment.
- Allow users to review the action plan before executing the code.
- At this stage, authentication credentials are used to connect to your infrastructure, if required.
> Note this command does not deploy anything, consider this as a dry-run command.

`terraform apply`:
- Deploys the instructions and statements in the code.
- Updates the state tracking mechanism file. (state file)

`terraform destroy`:
- Looks at the recorded, stored state file created during deployment and destroys all resources created by your code.
- Should be used with caution, as it is a non-reversible command.
- Take backups, and be sure that you want to delete infrastructure.

### Resource Addressing

#### Providers
aws example
```hcl
provider "aws" {
    region = "us-east-1"
}
```

gcp example
```hcl
provider "google" {
    credentials = file("credentials.json")
    project = "my-gcp-project"
    region = "us-west-1"
}
```

#### Resource blocks
```hcl
resource "aws_instance" "web" {
    ami = "ami-alb2c3d4"
    instance_type = "t2.micro"
}
```
Resource Address `aws_instance.web`

#### Data source blocks
```hcl
data "aws_instance" "my-vm" {
    instance_id = "i-123456789abcdefgh0"
}
```
Resource Address `data.aws_instance.my-vm`

### Terraform Code
- Terraform executes code in files with the `.tf` extension.
- Initially, Terraform looks for providers in the Terraform Providers Registry. Providers can also be sourced locally / internally and referenced with in your Terraform code.