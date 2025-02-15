# Using Packer to build a hardened AMI

The included packer files serve as a quick-start to creating an AMI in AWS.  
Currently included are three variants that can be used:   
- _**packer-ami.json**_: Creates a RHEL 7.7 instance, runs a simple Hello World script, and creates an AMI.
- _**packer-rhel7-ami.json**_: Creates a RHEL 7.0 instance, installs and runs openscap to harden the image, and creates an AMI. Scan results are pushed to an S3 Bucket.
- _**packer-centos-ami.json**_: Creates a CentOS 7 instance, installs and runs openscap to harden the image, and creates an AMI. Scan results are pushed to an S3 Bucket.

## Prerequisites:

### **AWS API and SSH Access**
You will need to have a keypair created for accessing AWS instances. If you do not already have a keypair, you can create one by following the steps described here: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html

AWS API access must be configured. This includes the appropriate .aws/config and .aws/credentials files to provide valid entries.  
Note: the profile name can be changed as needed, and multiple profiles can be defined allowing deployment to different accounts.   

### **Packer** 
Packer must be installed: https://www.packer.io/intro/getting-started/install.html

## Usage:

Set your AWS profile environment variables to define the profile configuration and credentials that will be used: 

### Environment variables for packer template: packer-ami.json
```
export AWS_ACCESS_KEY_ID='Your AWS access key ID'
export AWS_SECRET_ACCESS_KEY='Your AWS secret access key'
```

### Environment variables for packer template: packer-rhel7-ami.json
```
export REGION=us-east-1
export IAM_INSTANCE_PROFILE=hailstone-s3-upload
export BUCKET_NAME=hailstone-ami-scan-results
export INSTANCE_TYPE=t2.medium
export SSH_USERNAME=ec2-user
export AWS_ACCESS_KEY_ID='Your AWS access key ID'
export AWS_SECRET_ACCESS_KEY='Your AWS secret access key'
```

Check to verify that your packer file is properly formatted:  
```
packer validate packer-ami.json
```
You can also inspect your packer file to get an overview of the various components:  
```
packer inspect packer-ami.json
``` 
_Note: For the rhel/centos variants of the json file, you must reference the variables file when vaildating/inspecting as follows_ 
```
packer validate -var-file=variables.json packer-rhel7-ami.json
```

### Build sample ami including running the Hello World script
Build your new ami: 
```
packer build packer-ami.json
```

Packer will perform these steps: 
- create an instance from the specified RHEL 7.7 AMI
- connect via ssh to the instance
- run the `hello.sh` script
- stop the instance
- create AMI
- terminate the instance

The output will look like this: 
```
packer build packer-ami.json 
amazon-ebs output will be in this color.

==> amazon-ebs: Prevalidating AMI Name: rhel7-hardened-1568210354
    amazon-ebs: Found Image ID: ami-0916c408cb02e310b
==> amazon-ebs: Using existing SSH private key
==> amazon-ebs: Creating temporary security group for this instance: packer_<Sec Group ID>
==> amazon-ebs: Authorizing access to port 22 from [0.0.0.0/0] in the temporary security groups...
==> amazon-ebs: Launching a source AWS instance...
==> amazon-ebs: Adding tags to source instance
    amazon-ebs: Adding tag: "Name": "Packer Builder"
    amazon-ebs: Instance ID: <Instance ID>
==> amazon-ebs: Waiting for instance <Instance ID> to become ready...
==> amazon-ebs: Using ssh communicator to connect: <IP Address of instance>
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!
==> amazon-ebs: Provisioning with shell script: hello.sh
    amazon-ebs: Hello Kind World!
==> amazon-ebs: Stopping the source instance...
    amazon-ebs: Stopping instance
==> amazon-ebs: Waiting for the instance to stop...
==> amazon-ebs: Creating AMI rhel7-hardened-1568210354 from instance <Instance ID>
    amazon-ebs: AMI: <Image ID>
==> amazon-ebs: Waiting for AMI to become ready...
==> amazon-ebs: Terminating the source AWS instance...
==> amazon-ebs: Cleaning up any extra volumes...
==> amazon-ebs: No volumes to clean up, skipping
==> amazon-ebs: Deleting temporary security group...
Build 'amazon-ebs' finished.

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
us-east-1: <Image ID>

```

### Build a hardened RHEL/CentOS ami and publish the scan results to s3 bucket
```
packer build -var-file=variables.json packer-rhel7-ami.json
```   
or   
```
packer build -var-file=variables.json packer-centos-ami.json
```

After the successful execution of above command, you should see the new AMI created under the AMI section of the EC2 service on AWS.  
You should also see the scan reports under the S3 service of AWS within the bucket defined in your environment variables above (in this example `hailstone-ami-scan-results`).


===========================================

### To create a new hardened AMI

Whenever a Pull Request is created with reference to master branch, All checks should pass before one can review the code.

After tests are successful, we should see Jenkins build running in our CA Jenkins (Hailstone folder) which will do all of the above steps ( Create an EC2 instance, Connect via SSH, Run security scans (OpenSCAP - OVAL and STIG), Push the reports to S3, Stop the Instance, Create an AMI, Terminate an EC2 instance )


### NOTE -

When you add some documentation to README file and ssg-rhel7-ds-justifications.yaml, Jenkins will run a build and create an AMI in AWS.

### To Do -
A shell script can be created to launch packer build to ease the process. The script can also check  before build if any required variable is missing and guide use for the same.