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