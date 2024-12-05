# -AWS-CloudFormation-template
CIDR block for the VPC .
CIDR block for the public subnet . 
Amazon Machine Image ID for the Amazon Linux 2 AMI .

Parameters:
LabVpcCidr: The CIDR block for the VPC (default is 10.0.0.0/20).
PublicSubnetCidr: The CIDR block for the public subnet (default is 10.0.0.0/24).
AmazonLinuxAMIID: The Amazon Machine Image ID for the Amazon Linux 2 AMI (uses the AWS Systems Manager parameter for the latest Amazon Linux 2 AMI).
## Resources:
* LabVPC: A Virtual Private Cloud (VPC) is created with the provided CIDR block (10.0.0.0/20). DNS support and DNS hostnames are enabled.

* IGW: An Internet Gateway (IGW) is created to allow traffic between the VPC and the internet.

* VPCtoIGWConnection: This resource attaches the Internet Gateway to the VPC, enabling internet connectivity.

* MyS3Bucket: An S3 bucket is created.

* Ec2Instance: An EC2 instance is launched in the public subnet. The instance is configured to use the Amazon Linux AMI (AmazonLinuxAMIID), the t3.micro instance type, and is associated with a security group (AppSecurityGroup).

* PublicRouteTable: A route table is created for the public subnet.

* PublicRoute: A route is created within the route table to route all outbound traffic (0.0.0.0/0) to the Internet Gateway.

* PublicSubnet: A public subnet is created in the VPC using the provided CIDR block (10.0.0.0/24), and it is configured to automatically assign public IP addresses to instances launched in the subnet.

* PublicRouteTableAssociation: Associates the public route table with the public subnet, allowing instances in the public subnet to use the route.

* AppSecurityGroup: A security group is created to allow inbound HTTP traffic (port 80) from any IP address (0.0.0.0/0).

## Outputs:
LabVPCDefaultSecurityGroup: The default security group of the created VPC is outputted as part of the stack.

## Issues and Improvements:
1. Security Group Reference in EC2 Instance:

* The security group referenced by the EC2 instance (AppSecurityGroup) uses SecurityGroupIds. However, this can be problematic if the security group hasn't been created yet. Instead, you could reference the security group by its Ref value, ensuring it's created first.

    ```ymal
        SecurityGroupIds:
         - !Ref AppSecurityGroup

2. Missing InstanceProfile for EC2:

* If your EC2 instance needs access to AWS resources (like S3), you may need an InstanceProfile (IAM role) associated with the EC2 instance.

    ```ymal
      IamInstanceProfile: !Ref EC2InstanceProfile

3. S3 Bucket Permissions:

* If you're planning to interact with the S3 bucket (e.g., via EC2), make sure to define any required bucket policies or IAM roles/policies.
  
4. Availability Zone Selection:
* The Availability Zone selection uses !Select 0 !GetAZs Ref: AWS::Region, which is correct if you want to use the first AZ in the region. However, be cautious with hardcoding availability zone indexes if you expect to deploy in different regions or if you prefer high availability across multiple AZs.
  
5. Security Group Rule Enhancement:

* For enhanced security, consider restricting the inbound access to only trusted sources. Allowing 0.0.0.0/0 for port 80 may not be the most secure approach.
