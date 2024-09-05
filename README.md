# 1. VPC CloudFormation
## Usage

1. Login to your AWS account.
2. Provide the template URL:  
   [VELOCIS CFN](https://cfn.vsplcloud.services/)
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
echo -e '#!/bin/bash\necho "ECS_CLUSTER=Statging" >> /etc/ecs/ecs.config' | base64
```
## How to Retrieve the Latest ECS-Optimized AMI ID

To retrieve the latest ECS-optimized AMI ID for Amazon Linux 2023, use the following AWS CLI command:

```bash
aws ssm get-parameters --names /aws/service/ecs/optimized-ami/amazon-linux-2023/recommended/image_id --region ap-south-1 --query "Parameters[0].Value" --output text
```
[links for architechture type](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/retrieve-ecs-optimized_AMI.html)



# 3. ECS CloudFormation


This CloudFormation template creates an Amazon ECS cluster with associated capacity providers and a private DNS namespace. It allows you to configure various parameters to meet the specific requirements of your ECS environment.

## Parameters

The following parameters can be customized when creating or updating the stack:

### ECS Capacity Providers

- **AutoScalingGroupArn**:  
  _Description_: The ARN of the Auto Scaling group associated with the first capacity provider.  
  _Example_: `arn:aws:autoscaling:region:account-id:autoScalingGroup:autoScalingGroupName/xxxxxxxxxxxxx1`

- **AutoScalingGroupArn2**:  
  _Description_: The ARN of the Auto Scaling group associated with the second capacity provider.  
  _Example_: `arn:aws:autoscaling:region:account-id:autoScalingGroup:autoScalingGroupName/xxxxxxxxxxxxx1`

- **CapacityProviderName**:  
  _Description_: The name of the first ECS capacity provider.  
  _Default_: `FE`

- **CapacityProviderName2**:  
  _Description_: The name of the second ECS capacity provider.  
  _Default_: `BE`

- **CapacityProviders**:  
  _Description_: A comma-separated list of capacity providers for the ECS cluster.  
  _Example_: `FARGATE,FARGATE_SPOT,FE,BE`

### ECS Cluster Configuration

- **ClusterName**:  
  _Description_: The name of the ECS cluster.  
  _Default_: `DEMO`

### Private DNS Namespace Configuration

- **NamespaceName**:  
  _Description_: The name of the private DNS namespace for service discovery within the VPC.  
  _Default_: `demo.internal`

- **VpcId**:  
  _Description_: The VPC ID to associate with the namespace.  
  _Example_: `vpc-09449c77e104c511b`


## Outputs

After the stack creation is complete, the following resources will be available:
- An ECS cluster with the specified name and capacity providers.
- A private DNS namespace associated with the specified VPC.

## Notes

- Ensure that the provided ARNs for the Auto Scaling groups are correct and that the Auto Scaling groups are properly configured for use as ECS capacity providers.
- You can modify the parameters during stack creation or update to customize the ECS cluster configuration further.

# 4. ALB CloudFormation

This CloudFormation template creates an Application Load Balancer (ALB) with specified configuration options, including SSL certificates, subnets, and VPC associations. The template allows you to customize various parameters to suit your network and security requirements.

## Parameters

The following parameters can be customized when creating or updating the stack:

### ALB Configuration

- **CertificateArn**:  
  _Description_: The ARN of the SSL certificate to associate with the ALB.  
  _Example_: `arn:aws:acm:region:account-id:certificate/xxxxxxxxxxx`

- **LoadBalancerName**:  
  _Description_: The name of the Application Load Balancer.  
  _Default_: `DEMO`

- **Scheme**:  
  _Description_: The scheme of the ALB, specifying whether it is internet-facing or internal.  
  _Default_: `internet-facing`  
  _Options_: `internet-facing` or `internal`

- **Subnets**:  
  _Description_: A comma-separated list of subnet IDs where the ALB will be deployed.  
  _Example_: `subnet-0f1c6432b78501b25,subnet-0fdbd303a4af2afac`

### VPC Configuration

- **VpcId**:  
  _Description_: The ID of the VPC where the ALB and associated security group will be deployed.  
  _Example_: `vpc-09449c77e104c511b`

## Outputs

After the stack creation is complete, the following resources will be available:
- An Application Load Balancer with the specified name, scheme, and associated SSL certificate.
- The ALB will be deployed in the specified subnets within the provided VPC.

## Notes

- Ensure that the SSL certificate ARN provided is valid and associated with the correct region.
- The subnets provided must be part of the specified VPC.
- Modify the parameters during stack creation or update to further customize the ALB configuration as needed.

# 5. Elastic Load Balancer (ELB) Target Group (TG) CloudFormation Stack

This CloudFormation template creates an Elastic Load Balancer (ELB) Target Group (TG) with specified configurations, including listener rules, health checks, and target registrations. It allows you to customize various parameters to tailor the target group to your application's needs.

## Parameters

The following parameters can be customized when creating or updating the stack:

### Listener Rule Conditions

- **ConditionField1**:  
  _Description_: The first field for the listener rule condition (e.g., `path-pattern`, `host-header`).  
  _Default_: `path-pattern`

- **ConditionField2**:  
  _Description_: The second field for the listener rule condition (e.g., `path-pattern`, `host-header`).  
  _Default_: `host-header`

- **ConditionValues1**:  
  _Description_: The values for the first listener rule condition (e.g., `/*`, `*.example.com`).  
  _Default_: `/*`

- **ConditionValues2**:  
  _Description_: The values for the second listener rule condition (e.g., `/*`, `*.example.com`).  
  _Default_: `dev.example.com`

### Target Group Configuration

- **HealthCheckPath**:  
  _Description_: The path used for health checks on the target group.  
  _Default_: `/`

- **ListenerArn**:  
  _Description_: The ARN of the existing listener on port 443.  
  _Example_: `arn:aws:elasticloadbalancing:region:account-id:listener/app/xxxxxxxxxxxxx`

- **MatcherHttpCode**:  
  _Description_: The HTTP codes to use when checking for a healthy response from a target.  
  _Default_: `200-399`

- **Priority**:  
  _Description_: The priority for the listener rule.  
  _Default_: `10`

- **Protocol**:  
  _Description_: The protocol used by the target group.  
  _Default_: `HTTP`

- **TargetGroupName**:  
  _Description_: The name of the target group.  
  _Default_: `APP-TG`

- **TargetGroupPort**:  
  _Description_: The port on which the target group will receive traffic.  
  _Default_: `80`

- **TargetType**:  
  _Description_: The type of targets that you must specify when registering targets with this target group (instance, ip, lambda).  
  _Default_: `ip`

- **VPCId**:  
  _Description_: The VPC ID where the target group will be created.  
  _Example_: `vpc-09449c77e104c511b`


## Outputs

After the stack creation is complete, the following resources will be available:
- An Elastic Load Balancer (ELB) Target Group with the specified configuration.
- Listener rules with the defined conditions and priority.

## Notes

- Ensure that the `ListenerArn` provided corresponds to an existing listener.
- The `HealthCheckPath` should be a valid path that your targets can respond to for health checks.
- Modify the parameters during stack creation or update to further customize the target group configuration as needed.

# 6. ECS Service Launch Type EC2 CloudFormation

This CloudFormation template creates an ECS Service running on EC2 instances. It includes configurations for CloudWatch Logs, container settings, environment variables, and integration with Load Balancers and Service Registries.

## Parameters

The following parameters can be customized when creating or updating the stack:

### ECS Service Configuration

- **AwsLogsRegion**:  
  _Description_: The AWS region for CloudWatch Logs.  
  _Default_: `ap-south-2`

- **CapacityProvider**:  
  _Description_: The ECS Capacity Provider to be used.  
  _Default_: `FE`

- **Cluster**:  
  _Description_: The name of the ECS Cluster where the service will be deployed.  
  _Default_: `DEMO`

- **ContainerName**:  
  _Description_: The name of the container within the task definition.  
  _Default_: `FE`

- **ContainerPort**:  
  _Description_: The port on the container that the service will use.  
  _Default_: `80`

- **Cpu**:  
  _Description_: The CPU units allocated for the container.  
  _Default_: `100`  
  _Options_: `'256'`, `'512'`, `'1024'`, etc.

- **Environment**:  
  _Description_: Environment variables for the container.  
  _Default_: `Enter CommaDelimitedList`

- **EnvironmentFiles**:  
  _Description_: List of S3 URIs for environment files to be used by the container.  
  _Default_: `None`

- **ExecutionRoleArn**:  
  _Description_: The ARN of the Task Execution Role.  
  _Default_: `arn:aws:iam::267891377259:role/ecsTaskExecutionRole`

- **Image**:  
  _Description_: URL of the Docker image to be used in the container.  
  _Example_: `nginx`

- **Memory**:  
  _Description_: Memory (in MiB) allocated for the container.  
  _Default_: `128`  
  _Options_: `'512'`, `'1024'`, `'2048'`, etc.

- **RegistryArn**:  
  _Description_: ARN of the Service Registry for service discovery.  
  _Default_: `arn:aws:servicediscovery:ap-south-2:267891377259:service/srv-4ukjppecopecb56l`

- **SecurityGroups**:  
  _Description_: Security groups for the network configuration.  
  _Default_: `sg-01c8e1782b52bf3dd`

- **ServiceName**:  
  _Description_: The name of the ECS service.  
  _Default_: `FE`

- **Subnets**:  
  _Description_: A comma-separated list of subnet IDs where the service will be deployed.  
  _Default_: `subnet-0f1c6432b78501b25,subnet-0fdbd303a4af2afac`

- **TargetGroupArn**:  
  _Description_: The ARN of the Target Group for the Load Balancer.  
  _Default_: `arn:aws:elasticloadbalancing:ap-south-2:267891377259:targetgroup/FE-APP-TG/14dd65ff0a88b243`

- **TaskDefinitionFamily**:  
  _Description_: The ECS Task Definition Family and Name.  
  _Default_: `FE`

- **TaskRoleArn**:  
  _Description_: The ARN of the Task Role.  
  _Default_: `arn:aws:iam::267891377259:role/ecsTaskExecutionRole`

## Outputs

After the stack creation is complete, the following resources will be available:
- An ECS Service running on EC2 instances with the specified configurations.
- Integration with CloudWatch Logs, Service Registry, and Load Balancer.

## Notes

- Ensure that the provided `ExecutionRoleArn` and `TaskRoleArn` are valid and have the necessary permissions.
- Verify that the Docker image URL and environment variables are correctly configured for your container.
- The `TargetGroupArn` should be associated with a valid Application Load Balancer.##

