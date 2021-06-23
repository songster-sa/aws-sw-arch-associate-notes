## Contents
- [CIDR](#CIDR)
- [VPC creation](#VPC-creation)
- [nACL vs security groups](#nACL-vs-security-groups)
- [DNS resolution](#DNS-resolution)
- [VPC General](#VPC-General)
- [Connecting to a remote/corporate network](#Connecting-to-a-remote/corporate-network)
- [VPC Random](#VPC-Random)

## CIDR
- /32 = 2^0 = 1 IP only
- /24 = 2 ^ (32-24=8) = last part can change
- /16 = 2 ^ (32-16=16) = last 2 parts can change
- /0 = 2 ^ 32 = all IPs
- private IPs
    - 10.0.0.0/8     - for big networks
    - 172.16.0.0/12  - default AWS
    - 192.168.0.0/16 - home network

## VPC creation
- aws gives **default VPC** - all subnets are public / internet accessible - each instance has both public and private IP
    - if you delete by mistake its recoverable
- **Step 1** - creating VPC
    - u get option to use dedicated hardware or default (sharing) - "tenancy" option
    - creates route table, network ACL, security group
- **Step 2** - creating Subnet
    - subnet is associated with AZs
    - 1 AZ = 1 Private subnet (instances) + 1 Public subnet (load balancers only)
    - by default private - u assign public IP to make it public
    - aws reserves 5 addresses in ur CIDR range (256 becomes 251 available)
        - 10.0.0.0 - n/w address
        - .1 - router
        - .2 - DNS
        - .3 - future use
        - .255 - broadcast
- Step 2.5 - if you create an EC2 here, in ur public submet - wont be able to ssh as no internet
- **Step 3** - creating internet gateway - also serves as NAT
    - only 1 per vpc
    - by default detached - u need to attach to ur vpc
    - public SN will get access to internet
- **Step 4** - create new route table
    - should always leave the main / default route table private - so anything new create is not public
    - may have 1 private route table and 1 public route table
    - create route to igw - add outgoing public route (target is internet gateway) - and then associate subnets to it
    - 1 SN can be associated to only 1 route table at a time
- **Step 5** - NAT instances (outdated) vs NAT gateways
    - NATi
        - can be created by using apt ami - add to public SN - inc instance size if req
        - as its ec2, has a security group
        - must have elastic IP
        - disable the flag - src/dest checks
        - add to default/main/private route table - saying if someone wants to go out, use NATi as target
        - also add NATi to the public route table with target igw
        - DISADV - as its an EC2 instance - we will need to make it HA with ASG in multiple AZ and maintain and all
            - set inbound SG and outbound SG rules etc tec
    - NATg
        - create and add route to default/main/private route table
        - 1 per AZ - scales automatically
        - pay per HR for usage and bandwidth
        - ur ec2 can be across AZs..and connected to 1 NATg - but if that AZ goes down, all will loose internet access - so better have 1 NATg per AZ
- you need atleast 2 public subnets in order to create an internet facing ALB
    - ALB creation gives option if ALB is for internal use or internet facing
    - if u choose private SN for internet-facing - its a miss match

## nACL vs security groups
- network ACL - STATELESS - sit outside subnets - both inbound and outbound rules get evaluated
- SG - STATEFUL - if inbound passes, then outbound is not evaluated (and vice versa)
    - dont have deny rules
    - all rules are evaluated (when evaulated) - not by order or anything
- nACL - allow rules, deny rules, stateless
    - user created nacl - deny everything by default
    - aws created default nacl - allowed everything by default
    - 1 subnet can be associated with 1 nacl (vice versa is not true)
        - new subnets are assigned to default nACL by default
    - rules are evaluated in order (its first come first serve .. if u want to deny put if before allow)
    - as security groups dont have deny rules, to block any ip, its best to use nacl
- security group do not span across vpc - not visible outside ur vpc
- ephimeral ports - temp ports for communication(to/from) out of server

## DNS resolution
- enableDnsSupport
    - true by default
    - aws dns server at 169.254.169.253
- enableDnsHostname
    - false by default (true only in default vpc)
- when both true - assign public DNS hostname to any ec2 instance which has a public IP
- used when creating a "private hosted zone"

## VPC peering
- 2 VPC connected privately, over aws network
- not transitive connection
- can do across aws account, across region
- requires BOTH vpc's route table updates
- allows you to reference SG of peered VPC

## VPC endpoints
- connecting to aws resources need not leave aws n/w
- scale horizontally and redundant
- 2 types
    - interface endpoints (via ENI priv IP and SG) - used most of the times
        - for issues check DNS routing
    - gateway endpoints (via route table) - used for s3 and dynamo db
        - for issues check route table
- after creating, use --region in CLI command to access (region where you ec2 is)

## VPC General
- max 5 VPC per region -  soft limit
- max 5 CIDR per vpc - max /16 : min /28
- VPC is private - so Private IP ranges available
- should not overlap with you other/corporate LAN/private IPs
- internet gateway / VP gateway -> router -> route tables -> network ACL -> instances in public / private subnets via their security groups
- jump boxes / bastion hosts = ec2 instances in public SN via which u can ssh/rdp into instances in private SN
    - nothing to do on public instance - harden bastion host
    - just open access on private instance via security group (add inbound rules from public SN)
    - bad practice to save key-pair on public instance
    - can shh into - but priv still cant access internet
- flow logs
    - 3 types
        - VPC flow logs - includes the below 2
        - subnet flow logs
        - ENI flow logs
    - can go into s3 / cloud watch logs
    - to query them use - athena or cloud watch log insights
    - if u want flow logs for vpc peering, then both vpc have to be in same account
    - once flow log created - cannot change config later on
    - not logged - aws DNS, windows license server, 169.254.169.254, DHCP, reserved IP address

## Connecting to a remote/corporate network
- **Option 1 - Site to Site VPN**
    - VP gateway -> site to site vpn connection -> customer gateway
        - VPC console - customer gateway
        - VP gateway - attach to vpc
        - create VPN tieing VP gateway and customer gateway
        - setup the VPN on customer site / firewall
    - use customer gw's public IP or if its behind a NAT use NAT's public IP
- **Option 2 - Direct Connect DX**
    - dedicated private connection - not over public internet
    - you will still need a VP gateway on vpc outside
    - access aws resources (s3) and all ur vpc stuff
    - low cost, high bandwidth, real time data feeds
    - done via aws dx locations
    - client/customer router -> partner router -> aws DX router -> aws public / vpc private
        - in DX console - set up a public virtual interface or private virtual interface as required
    - connection types - takes atleast 1 month to set up
        - dedicated connections
        - hosted connections - hosted via partners - can add/remove capacity on demand
    - as its private - nothing is encrypted
        - but can do - by setting VPN on top of the connection
    - resiliency
        - high resiliency - set up DX via 2 DX locations - so if 1 goes down u can use other
        - MAX resiliency - set up DX via 2 DX locations - but in each location have 2 connections (so total 4)
- **Option 3 - Direct connect to multiple vpcs**
    - client/customer router -> partner router -> aws DX router -> DX gateway -> can direct to multiple vpcs
    - via private virtual interface

## VPC Random
- Global Accelerator
    - gives 2 static IPs or u can bring ur own
    - no ISP-DNS resolution to resolve aws urls
    - ur can use IPs or GA'S DNS name
    - network zone - similar to AZ
    - listeners (port, protocol, client affinity), endpoint grps per region (traffic dial), endpoints (ALB, EC2 - can have weighted endpoints as well)
- VPC private link - open up ur service (sitting in ur vpc) to 100s or other vpcs
    - one way - open to public via internet - bad for security
    - 2nd way - peering - will have to set up 100 peering connections
    - private link : ur vpc -> NLB -> ENI -> customer vpc
- transit gateway
    - simplify n/w arch / topology
    - transitive peering - hub-and-spoke model
    - regional, or cross region, or cross aws accounts
    - use route tables to control
    - works with DX , VPN connections
    - only aws service that supports IP multicast
- VPN cloud Hub
    - multiple sites with VPN among them and vpc connection via VP gateway
    - hub-and-spoke model
    - operates over public internet but all traffic is encrypted
- network costs
    - cross AZ is charged - within AZ is free
    - private IP is cheaper than public IP (going out to public internet and then coming in)
    - 1 vpc to another vpc is charged as well
    - all traffic coming in vpc is free
- NLB (with auto scaling) for multiple bastions - as they are for ssh/rdp = layer 4 - but this is expensive option
    - cheaper - 1 bastion with EIP - behind auto scaling of 1 min/max - then new instance will be created only when its lost and with same EIP
    - but will have some down time
    