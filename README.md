# [CloudBD CloudFormation All-In-One](https://www.cloudbd.io)

CloudBD CloudFormation All-In-One (`cfn-aio`) is a tool for easily testing CloudBD disks on Amazon Web Services using the AWS CLI and CloudFormation. `cfn-aio` will setup a self-contained environment that contains the AWS resources needed to use CloudBD disks such as a VPC, S3 Bucket and an EC2 instance. When you are done testing, `cfn-aio` can cleanup all the AWS resources it previously created.

## Prerequisites

* [**AWS CLI**](https://aws.amazon.com/cli/) on Linux or Mac (Windows is not supported at this time).
* [**CloudBD Account**](https://www.cloudbd.io) A free trial account can be obtained by signing up at [CloudBD](https://www.cloudbd.io).

## Quick Start

Follow the steps here to create your first CloudBD instance and disk in minutes.

1. **Clone this repository:**
  ```
  git clone https://github.com/dev-cloudbd/cloudbd-cfn-aio.git
  ```

2. **Install CloudBD credentials.json:**

  [Login](https://manage.cloudbd.io) and download your CloudBD credentials file. Then move the credentials file into the config directory:
  ```
  cd cloudbd-cfn-aio
  mv ~/Downloads/credentials.json config/
  ```

3. **Configure your default settings:**

  You may change the following default settings, or simply hit enter for each value to accept the default.
  ```
  ./cfn-aio configure
  Default AWS cli profile [default]:
  Default AWS region name [us-east-2]:
  Default remote name [remote]:
  Default availability zone letter in us-east-2 for EC2 [a]:
  Default IpV4 address range in CIDR format that may SSH in to EC2 instances [0.0.0.0/0]:
  Default instance type for EC2 [t3.small]:
  Default operating system for EC2 [amzn1]:
  ```

4. **Create and install AWS resources:**

  The following commands will create an S3 Bucket, VPC, IAM User, SSH key-pair and an EC2 instance using the configured default settings from the previous step.
  ```
  ./cfn-aio create-remote
  ./cfn-aio create-instance --name test1
  ```

  **Note:** AWS resources created above, such as the EC2 node and S3 bucket incur standard AWS charges for use and data storage.

5. **SSH into the EC2 instance:**
  ```
  ./cfn-aio ssh --name test1
  ```

6. **Create and attach a CloudBD disk:**

  After connecting to the instance with ssh you may use the following commands to create and attach your first disk named `testdisk`. If you changed the default remote name in step 3, please replace `remote` below with the remote name you set in the configure step.
  ```
  cloudbd create remote:testdisk 500G
  echo 'nbd0 remote:testdisk' | sudo tee -a /etc/cloudbd/cbdtab
  sudo cbddisks_start -a
  ```
  The above commands use the CloudBD CLI to create a disk and configures the disk to attach to the EC2 instance by adding an entry to the `/etc/cloudbd/cbdtab` file. The `cbddisks_start` command then starts the driver and attaches the disk to the server at `/dev/mapper/remote:testdisk`. Each entry in the `/etc/cloudbd/cbdtab` file must use a unique nbd number. Increment the nbd number for additional entries in the cbdtab file. Disks that are configured in the `cbdtab` file are automatically started upon reboot.

  More information can be found on these steps in the CloudBD documentation pages on [CLI - Disk Management](https://www.cloudbd.io/docs/gs-manage-disks.html), [Attach/Detach Disks](https://www.cloudbd.io/docs/dri-options.html), `man cbdtab`, and `cloudbd --help`.

7. **Format and mount a file system:**

  After attaching your first disk you may use the following commands to format it with ext4 and mount it to `/mnt`. If you changed the default remote name in step 3, please replace `remote` below with the remote name you set in the configure step.

  ```
  sudo mkfs.ext4 /dev/mapper/remote:testdisk
  echo '/dev/mapper/remote:testdisk /mnt ext4 _netdev,discard 0 0' | \
       sudo tee -a /etc/fstab
  sudo mount /mnt
  ```

  **The `_netdev` option is required for CloudBD disk entries in fstab to ensure the instance restarts correctly.** The `discard` option is recommended to free up unused storage on S3 when files are deleted. Additional detailed information about the above commands can be found in the [CloudBD documentation pages](https://www.cloudbd.io/docs/dri-options.html#filesystems).

  Your first CloudBD disk is now mounted and ready for use at `/mnt`. Data stored under `/mnt` is stored in an S3 bucket that was created with the `create remote` command in step 4. AWS S3 use and data storage charges apply to data stored on CloudBD disks.

## Quick Clean Up

  Perform the following steps to delete the data and clean up the resources created in the Quick Start above. These steps assume that you used the default remote name of `remote`, CloudBD disk name `testdisk` and mounted this disk at `/mnt`. Please adjust the commands to reflect any changes you may have made to these defaults.

1. **Unmount the file system:**

  Ssh into the CloudBD EC2 instance and unmount the CloudBD disk at `/mnt`.
  ```
  sudo umount /mnt
  ```

  Edit `/etc/fstab` and remove the line corresponding to the CloudBD disk mounted at `/mnt`.

2. **Detach and destroy the disk:**

  Edit the `/etc/cloudbd/cbdtab` file and remove the line corresponding to the CloudBD disk `remote:testdisk`. Perform the following commands to stop the disk and destroy it.

  **Note:** The `cloudbd destroy` command will **delete all data** stored on the disk. It is not possible to recover data after the disk has been destroyed.

  ```
  sudo cbddisks_stop remote:testdisk
  cloudbd destroy remote:testdisk
  ```

  More information can be found on these steps in the CloudBD documentation pages on [CLI - Disk Management](https://www.cloudbd.io/docs/gs-manage-disks.html), [Attach/Detach Disks](https://www.cloudbd.io/docs/dri-options.html), `man cbdtab`, and `cloudbd --help`.

3. **Delete AWS resources created by cfn-aio:**

  Prior to deleting the AWS resources, it is important to confirm that all CloudBD disks have been detached and destroyed. To confirm no CloudBD disks remain, execute the following command on the CloudBD EC2 instance.
  ```
  cloudbd list remote
  ```
  If the command output lists any disks, they will need to be unmounted, detached and destroyed prior to attempting to remove the AWS resources created by cfn-aio. Please repeat clean up steps 1 and 2 as needed to remove any CloudBD disks before proceeding on.

  When there are no CloudBD disks remaining, you can remove the CloudBD EC2 instance and the CloudBD remote with the following commands on the system where you initially cloned `cloudbd-cfn-aio`.
  ```
  ./cfn-aio delete-instance --name test1
  ./cfn-aio delete-remote
  ```

  Upon successful completion of the above commands, all AWS resources created by `cfn-aio` will have been destroyed.

## Additional Information

  `cfn-aio` supports multiple remotes and EC2 instances. For advanced usage of the `cfn-aio` tool, please see the built in help.
  ```
  ./cfn-aio help
  ```
