# 1. VPC CloudFormation
## Usage

1. Login to your AWS account.
2. Provide the template URL:  
   [VPC CloudFormation Template](https://cfn.vsplcloud.services/src/VPC.html)
3. Enter the required parameters based on your network configuration.
4. Review and confirm the settings, then create the stack.

This CloudFormation template creates a Virtual Private Cloud (VPC) with both public and private subnets across multiple Availability Zones. The template supports customization through various parameters, allowing you to configure the VPC according to your requirements.

## Parameters

The following parameters can be customized when creating or updating the stack:

### Global Configuration

- **Client Name**:  
  _Description_: Enter the client name.  
  _Default_: `Velocis`

### Network Configuration

- **Availability Zone**:  
  _Description_: Number of Availability Zones to utilize.  
  _Default_: `3`

- **Private Subnet**:  
  _Description_: Boolean value to indicate whether private subnets should be created in addition to public subnets.  
  _Default_: `true`

- **NAT Gateway**:  
  _Description_: Boolean value to indicate whether a NAT Gateway is needed.  
  _Default_: `Yes`

### Public VPC Configuration

- **VPC**:  
  _Description_: Enter your custom VPC CIDR block.  
  _Default_: `10.1.0.0/16`

- **Public CIDR 1A**:  
  _Description_: Enter the CIDR block for the first public subnet (Availability Zone 1a).  
  _Default_: `10.1.10.0/24`

- **Public CIDR 1B**:  
  _Description_: Enter the CIDR block for the second public subnet (Availability Zone 1b).  
  _Default_: `10.1.20.0/24`

- **Public CIDR 1C**:  
  _Description_: Enter the CIDR block for the third public subnet (Availability Zone 1c).  
  _Default_: `10.1.30.0/24`

### Private VPC Configuration

- **Private CIDR 1A**:  
  _Description_: Enter the CIDR block for the first private subnet (Availability Zone 1a).  
  _Default_: `10.1.50.0/24`

- **Private CIDR 1B**:  
  _Description_: Enter the CIDR block for the second private subnet (Availability Zone 1b).  
  _Default_: `10.1.60.0/24`

- **Private CIDR 1C**:  
  _Description_: Enter the CIDR block for the third private subnet (Availability Zone 1c).  
  _Default_: `10.1.70.0/24`



## Outputs

After the stack creation is complete, the following resources will be available:
- A custom VPC with the specified CIDR block.
- Public and private subnets across the specified Availability Zones.
- Optionally, a NAT Gateway if enabled.

## Notes

- Ensure that the CIDR blocks provided do not overlap with existing VPCs in your account.
- You can modify the parameters during stack creation or update to customize the network configuration further.

# 2. ASG CloudFormation

This CloudFormation template creates an Auto Scaling Group (ASG) with the specified configurations, including desired capacity, instance type, and security groups. The template allows you to customize various parameters to meet your specific needs.

## Parameters

The following parameters can be customized when creating or updating the stack:

### Auto Scaling Group Configuration

- **AutoScalingGroupName**:  
  _Description_: Name of the Auto Scaling Group.  
  _Default_: `FE-APP`

- **AvailabilityZone**:  
  _Description_: Availability Zone for the instances.  
  _Default_: `ap-south-2b`

- **DesiredCapacity**:  
  _Description_: Desired capacity of the Auto Scaling Group.  
  _Default_: `2`

- **IamInstanceProfile**:  
  _Description_: The name of the IAM instance profile role to attach to the instances.  
  _Default_: `AmazonEC2ContainerServiceforEC2Role`

- **ImageId**:  
  _Description_: The Amazon Machine Image (AMI) ID for the instances.  
  _Note_: You can retrieve the latest ECS-optimized AMI ID using the command provided below.

- **InstanceType**:  
  _Description_: EC2 instance type.  
  _Default_: `t3.medium`

- **KeyName**:  
  _Description_: The key name to use for SSH access to the instances.  
  _Default_: `test-apache-bench`

- **LaunchTemplateName**:  
  _Description_: Name of the Launch Template.  
  _Default_: `FE-APP-LT`

- **MaxSize**:  
  _Description_: Maximum size of the Auto Scaling Group.  
  _Default_: `5`

- **MinSize**:  
  _Description_: Minimum size of the Auto Scaling Group.  
  _Default_: `1`

- **SecurityGroup**:  
  _Description_: Security Group for the Launch Template.  
  _Default_: `sg-01c8e1782b52bf3dd`

- **UserData**:  
  _Description_: Base64-encoded UserData script for instance initialization.  
  _Default_: `IyEvYmluL2Jhc2gKZWNobyAiRUNTX0NMVVNURVI9TXlDbHVzdGVyIiA+PiAvZXRjL2Vjcy9lY3MuY29uZmlnCg==`

- **VPCZoneIdentifier**:  
  _Description_: List of subnet IDs for the Auto Scaling Group.  
  _Example_: `subnet-0f1c6432b78501b25,subnet-0fdbd303a4af2afac`

## How to get UserData
url to encode or decode
[url to encode or decode](https://www.base64decode.org/)
```
#!/bin/bash
echo "ECS_CLUSTER=Statging" >> /etc/ecs/ecs.config
```
linux command to encode base64

```
echo '#!/bin/bash\necho "ECS_CLUSTER=Statging" >> /etc/ecs/ecs.config' | base64
```
## How to Retrieve the Latest ECS-Optimized AMI ID

To retrieve the latest ECS-optimized AMI ID for Amazon Linux 2023, use the following AWS CLI command:

```bash
aws ssm get-parameters --names /aws/service/ecs/optimized-ami/amazon-linux-2023/recommended/image_id --region ap-south-1 --query "Parameters[0].Value" --output text
```
[links for architechture type](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/retrieve-ecs-optimized_AMI.html)




