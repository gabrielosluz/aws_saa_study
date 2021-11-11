# Notas AWS SAA 

## VPC

- Max 5 VPC per Region (soft limit).
- Cidr: min /28, max /16.
- Only IPv4 are allowed: 10.0.0.0/8; 172.16.0.0/12; 192.168.0.0/16.

## Subnet

- Subrange of IP.
- First 4 and last one are reserved. 

• Example: if CIDR block 10.0.0.0/24, then reserved IP addresses are:
10.0.0.0-Network Address
• 10.0.0.1 - reserved by AWS for the VPC router
• 10.0.0.2 - reserved by AWS for mapping to Amazon-provided DNS
• 10.0.0.3 - reserved by AWS for future use
• I0.0.0.255 - Network Broadcast Address. AWS does not support broadcast in a VPC, therefore the address is reserved.

## Internet Gateway & Route Tables

- Allow resources in a VPC to the internet. 
- One VPC to one IGW and vice versa. 
- Route tables also must be edited. 

## Bastion Host

- Use a BH to SSH into our private EC2 instances.
- The bastion is in the public subnet which is then connected to all other private subnets. 
- Make sure the bastion host has port 22 traffic from the IP address you need, not from the security groups of your other EC2 instance. 

## NAT Instances (outdated)

- NAT: Network Address Translation
- Allow EC2 instances in private subnets to connect the internet. 
- Must be launched in public subnet. 
- Must have a elastic IP.
- Route Tables must be configured to route traffic from private subnets to the NAT Instance.

## NAT Gateway

- Managed service. 
- High Availability.
- Pay per hour of usage and bandwidth. 
- Is created in specific AZ, uses an Elastic IP.  
- Requires a internet gateway to work. Private Subnet -> NATGTW -> IGW. 
- 5 to 45 GB. 
- Must create multiple NAT Gateway in multiple AZs for fault-tolerance. 

# DNS Resolution Options & Route 53 Private Zones

- DNS Resolution (enableDnsSupport): Decides if DNS resolution from Route 53 Resovler is supported for the VPC. 
- Default true. 
- DNS Hostnames (enableDnsHostnames). True -> Default VPC, False -> newly created VPCs. 
- If true, assigns public hostname to EC2 instance if it has a public IPv4. 

# NACLS & Security groups. 

- NACLs is stateless and Security Groups are statefull. 
- NACLs are like firewall wich control traffic from and to subnets. 
- One NACL per subnet, new subnets area assigned the Default NACL. 
- Rules 1 to 32766. 
- First rule match will drive the decision.
- Newly created NACLs will deny everything. 
- NACLs are a great way of blocking a specific IP Address at the subnet level. 
- Default NACL accepts everything inbound/outbound with the subnet it's associate with.
- Security groups supports allow rules only. NACL supports allow and deny rules. 

# VPC Reachability Analyzer. 

# VPC Peering

- Privately connect two VPC's using AWS's network.
- Make them behave as if they were in the same network. 
- Must not overlapping CIDR's. 
- You can create VPC Peering connection between VPC's in different AWS accounts/regions. 

# VPC Endpoint

- Private endpoints in VPC that allow you to iniciate a private connection to AWS Services. 
- Powered by PrivateLink.
- They remove the need of IGW, NATGW to access AWS Services. 
- In case of issues: check DNS Resolution in the VPC; check Route Tables. 
- Two types of endpoints: Interface Endpoints and Gateway Endpoints. 
- Interface endpoints: 
    - Provisions an ENI (private IP address) as an entrey point (must attach a security group).
    - Supports most AWS Service. 
- Gateway Endpoints:
    - Provisions a gateway and must be used as a target in a route table. 
    - Supports both S3 and DynamoDB.

# VPC Flow Logs

- Capture information about IP traffic going into your interfaces (VPC, subnets and ENI levels).
- Helps to monitor and troubleshoot connectivity issues. 
- Flow logs data can go to S3/CloudWatch Logs. 
- Capture network information from AWS managed interfaces too: ELB, RDS, ElastiCache, Redshift, NATGW, Transit Gateway etc...
- Query Flow Logs using Athena on S3 or CloudWatch Logs Insights. 

# Site to Site VPN, Virtual Private Gateway & Customer Gateway

- Site to Site VPN needs:
    - Virtual Private Gateway (VGW).
        -  VPN concetrator on the the AWS side of the VPN connection. 
        - VGW is created and attached to the VPC from which you want to create the site-to-site VPN connection. 
    - Customer Gateway:
        - Software application or physical device on customer side of the VPN connection. 

- Important step: enable Route Propagation for the Virtual Private Gateway in the route table that is associate with your subnets. 

- If you need to ping your EC2 instances from on-premise, make sure you add the ICMP protocol on the inbound of your security groups. 

- VPN CloudHub:
    - Multiple Customer Gateway. 
    - Provide secure communication between multiple sites, if you have multiple VPN connections. 
    - Low cost hub-and-spoke model for primary or secondary network connectivity between differente locations (VPN only). 
    - To setup it up, connect multiple VPN connections on the same VGW, setup dynamic routing and configure route tables. 

# Direct Connect and Direct Connect Gateway. 

- Provide a dedicated *private* connection from a remote network to your VPC. 
- Needs a Virtual Private Gateway on the VPC. 
- Direct Connect Gateway: if you want to setup a Direct Connect to one or more VPC in many different regions (same account).  
    - Dedicated host: 
        - 1 to 10 Gbps capacity. 
        - Physical ethernet port dedicated to a customer. 
        - Request made to AWS First. 
    - Hosted Connection: 
        - 50 Mbps, 500 Mbps, to 10 Gbps. 
        - Connection request made via AWS Direct Connect Partners. 
        - Capacity can be added or removed on demand. 
- Takes more than a month to establish new connections. 
- Data in transit is not encrypted but is private. 
- AWS Direct Connect + VPN provides an IPSec-encrypted private connection. 


# AWS PrivateLink - VPC Endpoint Services

- Most secure & scalable way to expose a service to 1000s of VPC (own or other accounts).
- Does not require VPC Peering, internet gateway, NAT, route tables etc. 
- Requires a network load balancer (Service VPC) and ENI (Customer VPC) or GWLB. 
- If the NLB is in multiple AZ, and the ENI's in multiple AZ, the solution is fault tolerant. 

# Transit Gateway

- For having transit peering between thousands of VPC abd on-premise, hub-and-spoke (star) connection. 
- Regional resource, can work cross-region. 
- Share cross-account using Resource Access Manager (RAM).
- You can peer Transit Gateway acrosss regions. 
- Route tables: limit which VPC can talk with other VPC. 
- Works with Direct Connect Gateway, VPN Connections. 
- Supports IP Multicast. 