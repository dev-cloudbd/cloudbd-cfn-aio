# [CloudBD CloudFormation All-In-One](https://www.cloudbd.io)

CloudBD CloudFormation All-In-One is a tool for easily testing CloudBD disks on Amazon Web Services using the AWS CLI and CloudFormation.

## Prerequisites

* [**AWS CLI**](https://aws.amazon.com/cli/) on Linux or Mac (Windows is not supported at this time).
* [**CloudBD Account**](https://www.cloudbd.io) A free trial account can be obtained by signing up at [CloudBD](https://www.cloudbd.io).

## Quick Start

Follow the steps here to create and deploy your first CloudBD instance in minutes.

1. **Clone this repository:**
  ```bash
  git clone https://github.com/dev-cloudbd/cloudbd-cfn-aio.git
  ```

2. **Install CloudBD credentials.json:**
  [Login](https://manage.cloudbd.io) and download your CloudBD credentials file. Then move the credentials file into the config directory:
  ```bash
  cd cloudbd-cfn-aio
  mv ~/Downloads/credentials.json config/
  ```

3. **Configure your default settings:**
  You may change the following default settings, or simply hit enter for each value to accept the default.
  ```bash
  ./cfn-aio configure
  Default AWS CLI profile [default]: 
  Default AWS region name [us-east-1]: 
  Default availability zone letter in us-east-1 for EC2 [a]: 
  Default IpV4 address range in CIDR format that may SSH in to EC2 instances [0.0.0.0/0]: 
  Default instance type for EC2 [t2.micro]: 
  Default operating system for EC2 [amzn1]:
  ```

4. **Create and Install AWS Resources:**
  The following commands will create an S3 Bucket, VPC, IAM User, SSH key-pair and an EC2 instance using the configured values.
  ```bash
  ./cfn-aio create-region
  ./cfn-aio create-instance --name test1
  ```

  **Note:** AWS resources created above such as the EC2 node and S3 bucket incure standard AWS charges for use and data storage.

5. **SSH into the EC2 Instance:**
  ```bash
  ./cfn-aio ssh --name test1
  ```

  Once connected to the EC2 instance, you may create and use CloudBD disks on the node. Please see the following page for information on how to create and manage CloudBD disks. [https://www.cloudbd.io/docs/gs-manage-disks.html](https://www.cloudbd.io/docs/gs-manage-disks.html)


