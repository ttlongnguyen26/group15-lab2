# Setup Instructions for EC2 Instance with AWS Cloud Formation

Here is the guide of lauching EC2 instances in Cloud Formation projects. The steps are presented sequentially below included generate an SSH key pair, configure your Terraform project, and launch your EC2 instance.

## Prerequisites

- Cloud Formation installed.
- AWS CLI installed for execute AWS functions through command line.
- AWS account with appropriate permissions to create EC2 instances, key pairs, and networking resources.
- SSH client to connect to the EC2 instance.

---

## A - Guidance of launch EC2 instance

### First
Change path into the Cloud Formation directory of this project as follow
```bash
cd "./NT548.P11_LAB1/Cloud Formation"
```

### 1. Create a S3 Bucket.

```bash
aws s3api create-bucket --bucket <s3_bucket_name> --region <your_region>
```
- When prompted, choose region following instructions of AWS [click here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html) (by default, choose "us-east-1" or "us-east-2").

- Your S3-bucket's name will must satisfy AWS constraints as follow this [link](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html). 

### 2. Generate a Key Pair

Use command `aws ec2 create-key-pair` to create a new RSA key pair that will be used for SSH access to your EC2 instance.

```bash
aws ec2 create-key-pair --key-name <your-key-pair-name>
```

### 3. Configure CloudFormation
This `parameter.json` file include parameters which construct VPC with Private and Public Subnet and EC2 instances with Security Group. 

***Cautions***: In this `parameter.json` file, you need to edit the path to the key pair name (create through the AWS EC2 instances) and your IP address for SSH access:

```json
    {
        "ParameterKey": "AllowedSSHIP",
        "ParameterValue": "<your-device-ip>" 
    },
    {
        "ParameterKey": "KeyPairName",
        "ParameterValue": "<your-keypair-name>"
    }
```

- Replace `your-keypair-name` with the name of your key pair name. Follow [AWS instructions](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-keypair.html#:~:text=To%20create%20a%20key%20pair&text=On%20the%20Key%20Pairs%20page,it%20in%20a%20safe%20place.) to create this.
- Replace `your-device-ip` with your current machine’s IP address (use [WhatIsMyIP](https://www.whatismyip.com/) or run `curl ifconfig.me` to find your public IP).
- [Optional] Beside that, you can edit some params in this project including: ```UniqueName```,  ```VpcCidr```, ```PublicSubnetCidr```, ```PrivateSubnetCidr```, ```AvailabilityZone```, ```InstanceType```


### Notes:

- The `keypair` is passed to Terraform to set up SSH access for the EC2 instance.
- Your IP address is used to limit inbound SSH access to only your device.
 

### 4. Initialize CloudFormation Nested Stack and Apply Configuration

Run the following commands to deploy VPC and EC2 instances:

```bash
# Check and verify the validity of the stacks
cfn-lint **/*.yml  

# Upload modules to S3 through S3 bucket 
aws s3 cp "./modules" s3://your-bucket-name/ --recursive --exclude "*" --include "*.yml" 

# Deploy the resources to AWS
aws cloudformation create-stack \
    --stack-name <your-stack-name>   \
    --template-body file://main.yml   \
    --parameters file://parameter.json   \
    --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND
    
# Check status of Stack
aws cloudformation describe-stacks --stack-name <your-stack-name>

# Destroy Stack after finishing deploy phase
aws cloudformation delete-stack --stack-name <your-stack-name>
```
### Notes:

- Check and add all necessary policies through AWS services.

## B - Guidance of Testing by Taskcat file

1. Install taskcat package through pip ***(if not installed yet)***
```bash
pip install taskcat 
```

2. Run below command (This code will run ```.taskcat.yml``` file in main project)
```bash
taskcat test run
```