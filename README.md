![High Level Design](Assets/AWS%20Shared%20VPC%20Endpoints-High%20Level%20Design.gif)

Centralizing VPC interface endpoints reduces deployment time, charges, and IP utilization across your AWS organizations. Interface endpoints (Powered by the AWS Private Link service) allow your workloads to reach AWS Services without an internet gateway or NAT gateway.  
# Endpoint Types
AWS has different types of VPC Endpoints with different functionalities. 
### Interface Endpoints (AWS Private Link) 
AWS [Interface Endpoints](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_interface_vpc_endpoints.html) place one or more Elastic Network Interfaces (ENI's) in a subnet. Each ENI is given a private IP either manually from the VPC CIDR Range or automatically by the AWS VPC. Traffic for AWS Services will go to these private IP's instead of the public IP's for the public service. 

Some of the key considerations of Interface Endpoints:  
- Private communication with AWS Services. 
- Can be controlled with Endpoint Policies & Security Groups.
- Resolves through Private DNS with route 53 from within the VPC. 
- Billed per hour, per AZ. Additional per GB data usage charges. 
- Can scale up to [100Gbps](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-limits-endpoints.html). 

### Gateway Endpoints
For supported services like S3, AWS maintains a managed prefix list containing the service's public IP address ranges. When you create a [gateway endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html), AWS adds a route to your route table with that prefix list as the destination and the endpoint as the target. Traffic bound for the service's public IPs is then carried over the AWS network, with no internet gateway or NAT required.

Some of the key considerations of Gateway Endpoints:  
- Currently supported for only AWS S3 and Dynamo DB. 
- No hourly data processing charge. 
- Traffic is routed over the AWS backbone to the "Public" endpoint for the services. 
- Uses no IP's from your subnets. 
- Controlled by endpoint policies. 

# The Problem with Interface Endpoints
![The Problem Design](Assets/AWS%20Shared%20VPC%20Endpoints-The%20Problem%20Design.drawio.png)

VPC Interface endpoints can be deployed into any AWS account. They require an ENI per AZ, per service type and are billed per hour per AZ per service type with an additional data usage charge. 
When deploying VPC Endpoints across multiple accounts, large amounts of IP space can be taken up, for example 4 accounts needing a bedrock runtime service in two AZ's would be a total of 8 IP's taken up and hourly charges. 



# The Solution: Centralized Interface Endpoints
Centralized VPC Endpoints can be deployed in a central network account and used by member accounts, reducing the amount of IP's, hourly charges, and endpoint policies across your organization. The example below describes the architecture. 

![Bedrock Usage](Assets/AWS%20Shared%20VPC%20Endpoints-Bedrock%20Usage.gif)
1. In this Example, A Lambda application requires private connectivity to the Bedrock Runtime for model invocations. 
2. Lambda is deployed into the VPC with a Security Group that allows access to the VPC Endpoints. 
3. Member account VPC's are attached to the centralized Transit Gateway. 
4. The traffic originating from the Lambda function is sent to the VPC Endpoint for invoking bedrock models.
5. The Bedrock Runtime endpoint traffic is sent over the AWS Network to the AWS Bedrock Service.  

## Detailed Architecture
The following architecture is a detailed conceptual reference architecture for how shared VPC endpoints can be utilized. 
![Final Conceptual Architecture](Assets/AWS%20Shared%20VPC%20Endpoints-Final%20Conceptual%20Architecture.gif)


## Reference Sources

- [Centralized Access to VPC Private Endpoints](https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/centralized-access-to-vpc-private-endpoints.html) - AWS Whitepaper on building scalable and secure multi-VPC network infrastructure
- [VPC Interface Endpoints](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_interface_vpc_endpoints.html) - AWS documentation on interface VPC endpoints
- [VPC Endpoint Limits](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-limits-endpoints.html) - AWS VPC PrivateLink service limits and quotas
- [Gateway Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html) - AWS documentation on gateway VPC endpoints
- [Sharing Managed Prefix Lists](https://docs.aws.amazon.com/vpc/latest/userguide/sharing-managed-prefix-lists.html) - AWS documentation on sharing managed prefix lists across accounts

