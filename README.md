# Terraform
A guide for learning/using Hashicorp Terraform.


## 1. Getting Started
- [Creating `main.tf`](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create#the-terraform-block)

used to define the **core configuration** of your infrastructure.

Think of it as the central blueprint or the starting point for our Terraform project. While we could spread our configuration across many `.tf` files, `main.tf` is a common convention

- [Adding a Cloud provider](https://registry.terraform.io/providers/hashicorp/aws/latest)

In this follow along, we will use AWS as our cloud provider. Instructions can be found in the [Terraform Registry](https://registry.terraform.io/browse/providers) 

> How to use AWS as a provider?<br>
To install AWS as a provider, copy and paste this code into your Terraform configuration. Then, run terraform init. Again, this can be found on the [Terraform Registry](https://registry.terraform.io/browse/providers) 

```tf
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "6.2.0"
    }
  }
}

provider "aws" {
  # Configuration options
}
```

- [Configuration Blocks](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create#configuration-blocks)
```tf
provider "aws" {
  region = "us-west-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  tags = {
    Name = "learn-terraform"
  }
}
```
> NOTE: All the code above can be found in the `main.tf` folder. Later we will execute them using the terminal

- [Configuration and credential files](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-format) 

Terraform interacts with AWS APIs to create, modify, or destroy resources (like EC2 instances, S3 buckets, or VPCs). To do this, AWS needs to verify who is making these requests and if they have the necessary permissions. The access key ID identifies the user or role making the request, and the secret access key acts as a cryptographic signature to authenticate the request, ensuring its legitimacy.

After getting the access key and secret access key from the AWS consol, we install the AWS CLI. This can be simply done by using `brew install awscli` on the terminal for macos users (considering you have [homebrew](https://brew.sh/) for the package manager). To then test the installation: `aws --version`

By typing `aws configure` we can write into `~/.aws/credentials` and `~/.aws/config` by following the given prompts

```
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]:
Default output format [None]:
```

> What is `~/.aws/credentials`? <br/>
A hidden file on your computer that safely stores your AWS access key ID and secret access key.
Simply put, it's where AWS tools like Terraform and the AWS Command Line Interface (CLI) look to find your "username" and "password" so they can connect to and manage your Amazon Web Services account on your behalf. In essence, **who you are**. Read more [here](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-where) 

> What is `~/.aws/config`? <br/>
It's a file for the AWS Command Line Interface (CLI) and AWS SDKs. It works in conjunction with ~/.aws/credentials to manage how your local machine interacts with your AWS accounts. In essence, **how and where you want** to work within AWS.

The format in `~/.aws/credentials` will look like this:
```
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```
And for `~/.aws/config`
```
[default]
region = us-east-1
output = json

[profile my-dev-profile]
region = us-west-2
output = text
```

NOTE: We can also write into `~/.aws/credentials` and `~/.aws/config` without using `aws configure`. This can be done by simply using `vim ~/.aws/credentials` to manually type.

## Terraform Lifecycle 
### [Formate Configuration](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create#format-configuration)
`terraform fmt` is a simple command that takes care of the "housekeeping" for Terraform files, making them organized, easy to read, and consistent across a team.
automatically reformats all configuration files in the current directory according to HashiCorp's recommended style.

### [(1/4) Initialize Terraform Project](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create#initialize-your-workspace)

Before you can apply your configuration, you must initialize your Terraform workspace with the `terraform init` command. As part of initialization, Terraform downloads and installs the providers defined in your configuration in your current working directory.

Terraform downloaded the `aws` provider and installed it in a hidden `.terraform` subdirectory of your current working directory. Terraform also created a file named `.terraform.lock.hcl` which specifies the exact provider versions used with your workspace, ensuring consistency between runs.

You will notice these folders come up after the command is ran.

### (2/4) Plan Terraform Changes
`terraform plan` command creates an execution plan that shows the changes Terraform will make to your infrastructure. This allows you to review these proposed changes to ensure they align with your expectations before you apply them.

### [(3/4) Validate Terraform Configuration](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create#validate-configuration)
`terraform validate` checks if the Terraform code is syntactically correct and internally consistent, without actually trying to create or change any real resources.  For example, if you mistype a resource name or refer to an argument your resource does not support, Terraform will report an error when you validate your configuration.

NOTE: Whenever we run `terraform apply` or `terraform plan`, it automatically runs  `terraform validate`

### [(4/4) Apply Terraform Configuration](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create#create-infrastructure)
by using `terraform apply`,
1. Terraform creates an execution plan for the changes it will make. Review this plan to ensure that Terraform will make the changes you expect.
2. Once you approve the execution plan, Terraform applies those changes using your workspace's providers.

This workflow ensures that you can detect and resolve any unexpected problems with your configuration before Terraform makes changes to your infrastructure.

NOTE: `terraform apply` creates a new file. `terraform.tfstate` 
the `terraform.tfstate` file is the backbone of Terraform's ability to manage your infrastructure reliably and efficiently. It's the persistent record that allows Terraform to understand, track, and reconcile the desired state of your infrastructure with its actual deployed state.

### [Inspect State](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create#inspect-state)
When you applied your configuration, Terraform wrote data about your infrastructure into a file called `terraform.tfstate`. Terraform stores data about your infrastructure in its state file, which it uses to manage resources over their lifecycle.

We can list the resources and data sources in our Terraform workspace's state with the `terraform state list` command.

Even though the data source is not an actual resource, Terraform tracks it in your state file. Print out your workspace's entire state using the `terraform show` command.

