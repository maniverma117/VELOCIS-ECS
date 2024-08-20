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

## License

This project is licensed under the MIT License.
