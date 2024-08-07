# Guide to Setting Up an AWS Web Server with Terraform

This guide walks you through the steps to create and bootstrap a web server on AWS using Terraform. It covers generating SSH keys and configuring Terraform files.

## Step 1: Configure AWS CLI

1. **Install the AWS CLI** (if not already installed):
   - Follow the installation instructions for your operating system [here](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

2. **Configure the AWS CLI**:
   ```sh
   aws configure
   ```

   You will be prompted to enter your AWS Access Key ID, Secret Access Key, Default region name, and Default output format. These credentials can be obtained from the AWS Management Console under IAM -> Users -> [Your User] -> Security credentials.

 **Example Output**:

```sh
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: json
```

## Step 2: Verify AWS CLI Configuration

Verify that your AWS CLI configuration is correct by running:
```sh
aws sts get-caller-identity
```

This command should return details about your AWS account and the IAM user or role you are using.

## Step 3: Generate SSH Key Pair

Generate an SSH key pair if you don't have one:

```sh
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
```

This command creates a private key (`id_rsa`) and a public key (`id_rsa.pub`) in the `~/.ssh/` directory.

>[!Note]
>When you generate an SSH key pair locally using `ssh-keygen`, it creates a private key (`~/.ssh/id_rsa`) and a public key (`~/.ssh/id_rsa.pub`). However, AWS needs to have a reference to this key pair to allow SSH access to the EC2 instances. The `aws_key_pair` resource uploads your public key to AWS and creates a key pair that AWS can manage.

## Step 4: Create Terraform Configuration Files

Create a directory for your Terraform project and navigate to it:

```sh
mkdir terraform-aws-webserver
cd terraform-aws-webserver
```

### Create `main.tf`

Create a `main.tf` file in your project directory with the following configuration:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_key_pair" "webserver-key" {
  key_name   = "webserver-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

data "aws_ssm_parameter" "webserver-ami" {
  name = "/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2"
}

resource "aws_vpc" "vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "terraform-vpc"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.vpc.id
}

data "aws_route_table" "main_route_table" {
  filter {
    name   = "association.main"
    values = ["true"]
  }
  filter {
    name   = "vpc-id"
    values = [aws_vpc.vpc.id]
  }
}

resource "aws_default_route_table" "internet_route" {
  default_route_table_id = data.aws_route_table.main_route_table.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  tags = {
    Name = "Terraform-RouteTable"
  }
}

data "aws_availability_zones" "azs" {
  state = "available"
}

resource "aws_subnet" "subnet" {
  availability_zone = element(data.aws_availability_zones.azs.names, 0)
  vpc_id            = aws_vpc.vpc.id
  cidr_block        = "10.0.1.0/24"
}

resource "aws_security_group" "sg" {
  name        = "sg"
  description = "Allow TCP/80 & TCP/22"
  vpc_id      = aws_vpc.vpc.id
  ingress {
    description = "Allow SSH traffic"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "Allow HTTP traffic"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "webserver" {
  ami                         = data.aws_ssm_parameter.webserver-ami.value
  instance_type               = "t3.micro"
  key_name                    = aws_key_pair.webserver-key.key_name
  associate_public_ip_address = true
  vpc_security_group_ids      = [aws_security_group.sg.id]
  subnet_id                   = aws_subnet.subnet.id

  provisioner "remote-exec" {
    inline = [
      "sudo yum -y install httpd && sudo systemctl start httpd",
      "echo '<h1><center>My Test Website With Help From Terraform Provisioner</center></h1>' > index.html",
      "sudo mv index.html /var/www/html/"
    ]
    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  tags = {
    Name = "webserver"
  }
}

output "Webserver-Public-IP" {
  value = aws_instance.webserver.public_ip
}
```

### Explanation of `main.tf`

- **`provider "aws"`**: Configures the AWS provider and region.
- **`resource "aws_key_pair" "webserver-key"`**: Creates an SSH key pair resource in AWS using your locally generated public key.
- **`data "aws_ssm_parameter" "webserver-ami"`**: Fetches the latest Amazon Linux 2 AMI ID using SSM Parameter Store.
- **`resource "aws_vpc" "vpc"`**: Creates a VPC with DNS support and hostname enabled.
- **`resource "aws_internet_gateway" "igw"`**: Creates an Internet Gateway for the VPC.
- **`data "aws_route_table" "main_route_table"`**: Fetches the main route table associated with the VPC.
- **`resource "aws_default_route_table" "internet_route"`**: Creates a default route table to route internet traffic through the Internet Gateway.
- **`data "aws_availability_zones" "azs"`**: Gets all available availability zones in the region.
- **`resource "aws_subnet" "subnet"`**: Creates a subnet in the first availability zone.
- **`resource "aws_security_group" "sg"`**: Creates a security group to allow HTTP and SSH access.
- **`resource "aws_instance" "webserver"`**: Defines an EC2 instance resource named `webserver`.
- **`provisioner "remote-exec"`**: Runs commands on the instance after launch to install and start the Apache HTTP server and set up a test webpage.
- **`output "Webserver-Public-IP"`**: Outputs the public IP address of the web server.

## Step 5: Initialize and Validate Terraform Configuration

1. **Initialize Terraform**:
   ```sh
   terraform init
   ```

2. **Validate the Configuration**:
   ```sh
   terraform validate
   ```

   This checks the syntax and consistency of the configuration files.

## Step 6: Plan and Apply the Configuration

1. **Generate and Review the Execution Plan**:
   ```sh
   terraform plan
   ```

   This creates an execution plan, showing what actions Terraform will take to create or modify infrastructure.

2. **Apply the Configuration**:
   ```sh
   terraform apply
   ```

   This will prompt you to confirm the plan. Type `yes` to proceed. Terraform will then create all the necessary resources, including the EC2 instance.

## Step 7: Access Your Web Server

Once the `terraform apply` command completes, it will output the public IP address of your web server. You can access your web server by navigating to this IP address in your web browser.

```sh
output "Webserver-Public-IP" {
  value = aws_instance.webserver.public_ip
}
```

### Example Output

```sh
Webserver-Public-IP = "54.83.98.202"
```

Visit `http://54.83.98.202` in your web browser to see your test website.

