# AWS Software Architect Associate Exam Notes 2020

Based on Udemy course by aCloudGuru (Ryan) and Stephane Maarek's course, own research and practise tests.
The topics on which most questions are based are highlighted in bold.

Notes here are incremental to my AWS Developer Associate exam notes :
https://github.com/songster-sa/aws-developer-associate-notes

## Contents
- [IAM](#IAM)
- [S3](#S3)
- [Data Sync](#Data-Sync)
- [Cloud Front](#Cloud-Front)
- [SnowBall](#SnowBall)
- [Storage Gateway](#Storage-Gateway)
- [Athena vs Macie](#Athena-vs-Macie)
- [EC2](#EC2)
- [EBS](#EBS)
- [EFS](#EFS)
- [HPC - high performance compute architecture](#HPC---high-performance-compute-architecture)
- [RDS](#RDS)
- [DynamoDB](#DynamoDB)
- [Redshift](#Redshift)
- [Aurora](#Aurora)
- [DMS - database migration system](#DMS---database-migration-system)
- [Amazon EMR - elastic map reduce](#Amazon-EMR---elastic-map-reduce)
- [AWS Directory Service](#AWS-Directory-Service)
- [AWS Resource Access Manager](#AWS-Resource-Access-Manager)
- [AWS SSO](#AWS-SSO)
- [VPC](#VPC)
- [On-premise strategies](#On-premise-strategies)
- [Elastic transcoder](#Elastic-transcoder)
- [Reduce security threats](#Reduce-security-threats)
- [Lambda](#Lambda)
- [Random points](#Random-points)

## IAM
- policies - effect, action, resource
    - identity policy - attached to iam user/roles/grps
    - resource policy - attached to resources
- permission boundaries - delegate admin
    - can be set only for IAM users/ roles
    - dont allow or deny - simply define the max perm entity can have
    - if present then this takes priority and is final

## S3
- components - key, value, versionId, metadata, tiers, lifecycle mgmt, versioning, encryption
- security - ACL , torrent ? (bucket policy)
- MFA delete
- Glacier standard vs Deep archive
- Cross Region Replication 
    - can be configured via lifecycle rules
    - versioning should be on in both src and dest bucket
    - can be set for 1 bucket or all buckets
    - applies to objects stored AFTER its set ie new versions (old / existing versions are not replicated)
    - if src is made public, dest is NOT made public
    - if src object is deleted - dest object is NOT deleted
- Transfer Acceleration
    - u get distinct url to use (to upload via edge locations)
    - google a tool to test it
- costs for tier - high to low - standard, IA, Intelligent, one-zone IA, Glacier, deep archive
- WORM lock - 2 modes
    - governance mode - can do actions with permissions
    - compliance mode - no one can do anything, not even root
- Glacier Vault Lock (policy)
- more prefixes (folder) we have in S3 path - better performance
    - n x 5500 = GET per second
    - n x 3500 = PUT/COPY/DELETE per second
- 3 ways to share access across S3
    - policy + IAM = bucket level = programmatic access
    - ACL + IAM = object level = programmatic access
    - cross account IAM roles = prog + console access
- lifecycle policies - apply to old and new objects and versions
- limit of 100 buckets per account
- 4 URL styles
    - virtual host
    - path style
    - Static web
    - legacy global endpoint

## Data Sync
- agent on on-premise system
- used to share / transfer NFS or SMB compatible FS to AWS (s3,EFS,FSx for windows)
- automatic encryption - both at-rest and in-transit
- automatic integrity checks
- replication per hr/day/week
- can also be used on EC2 - for EFS to EFS

## Cloud Front
- origins can be - S3, EC2, Route53, ELB
- u can clear cache - but its charged
- distribution - 2 types
    - web
    - RTMP - for adobe flash media
- u can restrict access via signed urls only - data not accessible directly
- disable a distribution before deleting it
- 1 signed url -> access 1 files
- 1 cookie -> access many files
- signed url key-pair managed by root account
- policies like - URL expiration, IP ranges, trusted signers
- Diff with S3 signed url - S3 one is temporary and for direct access, created by IAM user

## SnowBall
- big disk - peta-byte - 50-80TB
- security -  tamper resistant, encryption, TPM (trusted platform module)
- after transfer aws erases it using software
- SnowBall Edge - 100TB - compute + storage - offline aws??
- SnowBall Mobile - exa-byte
- SnowBall Client - to export data
- can use S3 to import-export

## Storage Gateway
- is hardware or software
- transfer from on-premise(site) to aws
- via direct connection / internet / VPC
- 3 types
    - file gateway - for NFS / SMB - direct on S3
    - volume gateway 
        - stored as EBS volume snapshots (incremental)
        - compressed and stored on S3
        - 2 sub types
            - stored volume - all data on site, backup on S3
            - cached volume - all data on S3, only freq used on site

## Athena vs Macie
- Athena 
    - SQL - query data on S3
    - pay per query
    - serverless
    - for logs, reports, click stream data
- Macie
    - security service to descover, classify and protect PII
    - use machine learning and NLP, AI
    - use cloud trail logs
    - creates dashboards, reports and alerts

## EC2
- Spot instances - huge saving 90%
    - if aws stops then u r not charged for the partial hr - based on spot price - 2 min to decide stop or terminate
    - Spot block - 1-6hr block
    - Spot Request - one time or persistent
    - Spot fleet - spot instances + on demand instances
- Spot instance strategies for the fleet
    - capacity optimised
    - lowest price - default
    - diversified - distribute across all launch-pools defined
    - instance pools to use count
- EC2 instance connect - tool
- Status checks
    - system status check
    - instance status check
- termination protection - can be set
- all volumes (inc root) - encryption can be set, delete-on-termination can be set
- Security groups - STATEFUL
- 1 EC2 can have many Security groups
- ENI - elastic network interface - when you need separate n/w 
- EN - enhanced networking - when you need very good network performance / speed
    - VF - <10 Gps
    - ENA - 10-100Gps
- EFA - elastic fabric adapter - for HPC + machine learning - for OS bypass
- EC2 hibernate (diff between stop , terminate, hibernate)
    - saves RAM to root vol - root vol should be encrypted
    - OS is not stopped
    - for on-demand and reserved instances
    - max 60 days
    - RAM <= 150GB
    - available for only C, M, R instances
- Placement groups
    - all PGs have to be in same region
    - name has to be unique within ur account
    - cannot merge PGs
    - can move an outside ec2 inside - STOP + CLI or SDK - confirm?
    - types
        - cluster - all in same AZ - high n/w throughputs - only available for high instance types
        - spread - each instance on diff hardware - 7 max per AZ - can span across AZ
        - partition - mix - each partition on diff hardware but 1 partition can many many ec2 - 1 AZ max 7 partition 
            - can span across AZ ????
            - for distributed databases like cassandra,kafka, hadoop etc
- Instance types
    - R - RAM - in-memory caches
    - C - CPU - compute / databases
    - M - middle - general / web app
    - I - I/O - instance store / databases
    - G - GPU - video rendering / machine learning
    - T2 T3 burstable
    - T2 T3 unlimited burst - but u pay for it
- AMIs - region specific - by default all private and live in S3
    - u can share with other accounts - u own ami
    - if they copy - they own the copied ami (add create-volume permission checkbox)
    - to copy 
        - u need access to backed storage ( EBS or S3 )
        - OR - u launch instance and then create new AMI from that 
    - encrypted AMI - cant copy 
        - share storage snapshot and encrypted key - copy this snapshot and re-encrypt with own key
        - then register as new AMI
    - billing product AMI - cant copy - launch instance and create AMI form there
  
## EBS
- gp2 , io1, st1, sc1, standard
- root vol is always in same AZ as ec2 - comes from ami snapshot
- u can change size/storage of vols on the fly - take some time
- how to move vol to another AZ - 
- how to move vol to another region -
- can detach EBS on fly or after stopping ?
- snapshots 
    - on S3, always incremental
    - can take on fly or after stopping
- Instance Store (INS) 
    - has to be added at ec2 creation time - cannot add later
    - volume dashboard wont show
    - can either reboot or terminate
    - cannot stop - if stopped due to issue then data lost
- how to create encrypted vol from unencrypted vol
- u can share only unencrypted vols
- 1 EBS can be attached to many EC2 - possible since feb 2020

## EFS
- amazon-efs-utils tool to mount on various ec2s
- add NFS 2049 on security groups
- lifecycle tiers - to move to IA if required
- encryption - both at-rest and in-transit
- support NFS v4 protocol
- read after write consistency
- applies within region (across AZ and ec2)
- linus only
- Diff with FSx Windows - its SMB based, windows based, distributed FS
- FSx Lustre - for HPC , machine learning, EDA, millions of IOPs, can store data directly on S3

## HPC - high performance compute architecture
- SnowBall , SnowMobile, DataSync, DirectConnect
- ec2 instance, ec2 fleet, placement group (cluster)
- ENA, EFA
- EBS with provisioned IOPs , instance store
- S3, EFS with provisioned IOPs, FSx Lustre
- aws batch - schedule, parallel , multiple jobs
- parallel cluster - automate creation of VPC, simple text file for provisioning

## RDS
- has types of instances ?
    - reserved instance
- multi az - for disaster recovery - automatic fail over - no dns url change
    - u can force failover from one AZ to another by rebooting the RDS instance
- read replicas - for high performance - no automatic fail over - u have to create new DNS url and then use it
    - u must have automatic backup turned on - to create Read replicas
    - RRs can have multi AZ of their own
    - RRs can be in other regions
    - not for MS SQL
    - replication is free?
- have option for storage auto scaling - to add storage on the fly
- have option of termination protection and maintenance window
- RDS runs on VMs, but u dont have access to those VMs
- backups storage - u get same size in S3 as ur DB size
- when u restore - it will be a new RDS instance with new endpoint
- once encryption is not - everything related will be encrypted (backups, snapshots, read replicas, data)
- mySQL installation default port - 3306

## DynamoDB
- stored on SSD - spread across 3 geo distinct data centers
- Transactions - prepare+commit - upto 25 items or 4MB of data at a time
- capacity
    - provisioned - 
    - on demand - pay per request - still scales up automatically - but u pay for storage and backup not for read/writes
        - good for new apps where we dono the expected load
- Streams - can be used for cross region replication, events-Lambda, relationships across table
    - global tables - use streams to replicate
- auto encryption at rest - of course via KMS

## Redshift
- single node - 160gb
- multi node = leader node + upto 128 compute nodes
- adv compression - on columns
- does not need indexing
- massive parallel processing
- backups - 1-35days (1 default)- enabled by default
- maintains 3 copies of data - original, replica, backup(s3)
- can async replicate cross region for DR in S3
- charge for compute node hours only + backups and any data transfer
- encryption - in transit + at rest
- no multi AZ - but if outage - u can use snapshots to restore in diff AZ

#Aurora
- starts with 10GB, auto scales in 10GB increments - upto 64TB
- compute resources can scale upto 32vCPU and 244GB
- auto backups always enabled - dont affect performance
- can share snapshots with other aws accounts
- aurora serverless 
    - on demand, cost effective
    - automatically starts, shuts down, scales up or down
    - for infrequent, intermittent, unpredictable workloads
- how to convert MYSQL to Aurora - create aurora read replica - then promote to DB (or take snapshot of RR and then restore as new DB)
    - can do in a diff region
    - writer node - diff endpoint - diff AZ
    - reader node - diff endpoint - diff AZ

## DMS - database migration system
- src (on-premise, ec2, rds, s3, azure sql) - dest (on-premise, ec2, RDS etc) - can be any DB
- src remains operational during migration
- is a server in cloud that runs replicate s/w
- how
    - create src and dest connections
    - schedule task on the DMS server
- u can let DMS create tables on dest, or u can pre-create them (manually or use SCT - schema conversion tool , some or all)
- supports homogeneous and heterogeneous (need SCT) migrations

## Amazon EMR - elastic map reduce
- big data platform
- peta byte scale analysis
- cluster of ec2 nodes - each node has own role to play, own s/w installed
- node types
    - master node - manages cluster, track tasks, health check
    - core node - run tasks and store data - mandatory
    - task node - run tasks but not store data - optional
- each node talks to each other
- logs is on master node - u need to periodically archive to S3 to ensure u dont loose it in case the node dies
    - this way logs will be there even after cluster terminates - otherwise lost
    - EMR archives every 5min
    - u can set it up ONLY when u first create cluster - not later

## AWS Directory Service
- connect aws resource to on-premise active directory (AD)
- its a standalone directory in cloud
- helps u access aws using corporate credentials
- u can SSO to all ec2 instance joined to your AD domain - using AD Connectors (directory gateway)
- What is AD - users, groups, policies, LDAP and DNS, kerberos auth, high availability
- AWS managed Microsoft AD Domain Controllers - run on windows - in multi AZ (default 2)
    - extend aws AD to on-premise AD using AD Trust
    - u have to take care of only the data eg users, groups, policies, scaling DC, AD Trust, LDAPS (certificate authority), federation
    - aws takes of - multi AZ, patching, snapshot and restore, instance rotation
- Simple AD
    - basic AD features
    - small <=500 users, large <=5000 users
    - does not support AD Trust
- CLoud Directory - fully managed directory-based store for developers
    - for apps which create/use/work with org charts, catalogs, registries
    - not AD compatible

## AWS Resource Access Manager
- share resource access across aws accounts
- App Mesh, Aurora, CodeBuild, EC2, EC2 image builder, License manager, Resource Groups, Route 53
- use RAM in account 1 - share - go in RAM account 2 - accept the invitation to share
- only share access - cannot change the resource - so clone DB in own account to use

## AWS SSO
- SSO for all aws accounts + external apps (office, salesforce etc)
- integrate with on-premise AD, or any SAML provider
- all logins recorded in Cloud Trail
- use existing identities to login various apps

## Route 53
- top level domain server -> name server -> start of authority server -> DNS record -> IP
- default TTL is 48hr with most ISP
- cant have CNAME for naked domain names (without www)
- MX record
- PTR record - like WhoIs - lookup IP to find domain name
- u can buy domains directly with amazon - it gives multiple top level domains to you - in case one goes down
- u can set health checks - if fails then route is taken out until it passes
- Geo-proximity routing - traffic flow only - create traffic policies
    - geo based + bias setting

## VPC
- network ACL - STATELESS
- internet gateway / VP gateway -> router -> route tables -> network ACL -> instances in public / private subnets via their security groups
- jump boxes / bastion hosts = ec2 instances in public SN via which u can ssh/rdp into instances in private SN
    - nothing to do on public instance - harden bastion host
    - just open access on private instance via security group (add inbound rules from public SN)
    - bad practice to save key-pair on public instance
    - can shh into - but priv still cant access internet
- aws allows max /16 SN CIDR - /28 is the smallest
- aws gives default VPC - all subnets are public / internet accessible - each instance has both public and private IP
    - if you delete by mistake its recoverable
- when creating VPC
    - u get option to use dedicated hardware or default (sharing)
    - creates route table, network ACL, security group
- when creating SN - by default private - u assign public IP to make it public
- when creating internet gateway - by default detached - u need to attach to ur vpc
    - only 1 per vpc
- should always leave the main / default route table private - so anything new create is not public
    - create new route table - add outgoing public route (target is internet gateway) - and then associate subnets to it
    - 1 SN can be associated to only 1 route table at a time
- security group do not span across vpc - not visible outside ur vpc
- NAT instances vs NAT gateways
    - NATi 
        - can be create by using apt ami - add to public SN - inc instance size if req
        - as its ec2, has a security group
        - disable src/dest checks
        - add to default/main/private route table - saying if someone wants to go out, use NATi as target
    - NATg - create and add route to default/main/private route table 
        - 1 per AZ - scales automatically
        - ur ec2 can be across AZs..and connected to 1 NATg - but if that AZ goes down, all will loose internet access - so better have 1 NATg per AZ
- nACL - allow rules, deny rules, stateless
    - user created nacl - deny everything by default
    - aws created default nacl - allowed everything by default
    - 1 sub can be associated with 1 nacl (vice versa is not true)
    - rules are evaluated in order (its first come first serve .. if u want to deny put if before allow)
    - ephimeral ports - temp ports for communication out of server
    - as security groups dont have deny rules, to block any ip, its best to use nacl
- you need atleast 2 public subnets in order to create an internet facing ALB
    - ALB creation gives option if ALB is for internal use or internet facing
    - if u choose private SN for internet-facing - its a miss match
- flow logs 
    - if u want flow logs for vpc peering, then both vpc have to be in same account
    - once flow log created - cannot change config later on
    - not logged - aws DNS, windows license server, 169.254.169.254, DHCP, reserved IP address
- direct connect : client router -> partner router -> DX router -> aws public / vpc private
    - DX console - public VI
    - VPC console - customer gateway
    - VP gateway - attach to vpc
    - create VPN tieing VP gateway and customer gateway
    - setup the VPN on customer site / firewall
- Global Accelerator
    - gives 2 static IPs or u can bring ur own
    - no ISP-DNS resolution to resolve aws urls
    - ur can use IPs or GA'S DNS name
    - network zone - similar to AZ
    - listeners (port, protocol, client affinity), endpoint grps per region (traffic dial), endpoints (ALB, EC2 - can have weighted endpoints as well)
- vpc endpoints - connecting to aws resources need not leave aws n/w
    - 2 types - interface endpoints (via ENI priv IP) ; gateway endpoints (s3 and dynamo db)
    - after creating, use --region in CLI command to access
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

## On-premise strategies
- DMS - db migration service
- SMS - server migration service
    - incremental replication of on-prem server
- application discovery service
    - helps plan the migration
    - install "app discovery agentless connector" - on to ur vmware
    - it will create (server utilization and dependency) map of everything - store encrypted
    - data also available to "migration hub"
- VM import/export
    - app to ec2 and vice versa
- can download your amazon linux2 as ISO - take ec2 and run on prem

## Elastic transcoder
- media transcoder - convert media formats
- pay by minute and resolution
- s3 -> lambda -> transcoder -> another s3 bucket

## Reduce security threats
- ALB
    - NACL -> ALB -> ec2 (coz client connection terminates at ALB so ec2 wont get to know)
    - WAF -> ALB -> ec2
- NLB
    - NACL -> NLB -> ec2 (NLB does not block connection - so IP add is visible end to end)
- cloud front
    - WAF -> cloud front -> NACL -> ALB -> ec2 (againt the conn terminates at CF - so use WAF)
- NACL only deals with IPs and WAF handles other things as well (XSS, SQL inj etc) - so choose accordingly
- KMS - FIPS 140-2 level 2
    - once u encrypt file - when u decrypt u dont need to mention key/alias again as its kept in the metadata
    - u will always have to decode base64
    - --key-id "alias/acgDemo"
    - multi tenant - uses cloud HSM internally but HSM is shared
- cloud HSM - FIPS 140-2 level 3 - dedicated HSM
    - manage ur own keys
    - single tenant multi AZ
    - standard industry API - no aws APIs - PKCS#11 / JCE / CNG
    - aws is not aware of ur keys - u need to keep them safe
    - u create one HSM per AZ (for DR) and HSM exposes ENI - which u use to communicate
- SMPS - component of Systems manager
    - manages configs and secrets - caches and distributing to aws resources
    - serverless - scalable
    - can be encrypted KMS
    - hierarchy, version, TTL
    - grant access to hierarchy tree - a leaf or level etc - 15 levels deep - GetParametersByPath
    - integrated with cloud formation
    - standard - limit of 10k params, value size 4kb
    - advances - > 10k, 8kb
    - types - string, string list, secure string
    
## Lambda
- if 10 users trigger lambda - 10 functions run - thats how it scales automatically (1 event = 1+ function)
- first 1 million requests are free - after than 0.20$ per 1 million requests
- for usage - u r charged per GB-second usage (ie how much memory u allocated + whats the duration of the function)
- what can trigger lambda - 
    - API gateway , IOT, ALB
    - cloud watch events, logs
    - code commit
    - congnito sync
    - dynamodb, kinesis
    - S3, SNS, SQS
    - alexa skill
- what cant trigger lambda - RDS
- can use cloud trail, x-ray 
- lambda is global
- SAM - cloud formation extension for serverless
    - sam init, sam build, sam deploy --guided

## Random points
- aws account creation - support types 4
    - basic , developer, business, enterprise
- every aws account does gets its own dedicated default VPC
- can create billing alarm via cloud watch
    - threshold configured per hr or month etc
    - SNS notification
- global services - IAM, S3, CloudFront, 
- can make architecture super secure by using multiple aws accounts
- AWS Organization
    - paying account
    - SCP (service control policy) - to enable/disable services on accounts
- using SAML u can give ur federated users SSO access to AWS mgmt console
- IAM power user - access to all services except IAM users, groups
- when creating ami - virtualization has 2 options
    - PV
    - HV - better to use - can use to launch diff ec2 types
- when u choose ami to launch ec2 - u can choose based on many things
    - OS, region, AZ, architecture, launch permissions
    - root vol type - EBS or instance store (INS)
    - for EBS - snapshot on S3
    - for INS - template on S3
- any serverless pricing is always by use (invocation) - otherwise its by hr/minute
- who has caching - cloud front, api gateway, elastic cache, DAX
    - the more caching u provide in the arch in the front, the less load on DB or backend
- ARN - arn:partition:service:region:account_id:resource_type/resource
- aws randomises AZ name - so my 2a may be diff from ur 2a
- aws reserves 5 addresses in ur CIDR range (256 becomes 251 available)
    - 10.0.0.0 - n/w address
    - .1 - router
    - .2 - DNS
    - .3 - future use
    - .255 - broadcast
- common port numbers - 80,443,22, 3306(aurora)
- every ec2 instance has a check to confirm its either src or dest of traffic
- ASG components - groups, config templates, scaling options
    - can define lifecycle hooks at pending-wait / termination-wait
    - when removing an instance - 1st the zone with max instances is chosen, then the instance with oldest launch config is terminated
- scaling options
    - maintain current levels(no of instance) all time
    - scale manually - mention min/desired capacity
    - scale on demand - define params to control
    - scale on schedule
    - scale on prediction
- when making HA arch :
    - aws s3 cp --recursive /var/www/html s3://bucket-name
    - .htaccess file - contains url rewrite to route meant for s3 to go to cloudfront
    - aws s3 sync /var/www/html s3://bucket-name
    - tell httpd that we allow url rewrite
    - for auto polling - write command in /etc/crontab file
- test RDS failover by simply rebooting 
- Quick Start - has bunch of cloud formation templates for various use cases
- elastic beanstalk - no infra, no template - just upload code and it will figure out what to do and provision everything
    - all auto scaling etc can be done from its UI
- SQS - used to decouple components
- same origin policy - is done by browser to prevent cross-site-scripting attacks
    - but in aws all resources have own domains - so allow CORS - for somethings like fonts
    - enabled at server side (not client or browser side) - but enforced by client / browser
    - HTTP OPTIONS - there are the domain approved - error = origin policy cannot be read at the remote resource = then allow CORS
- so when you create a key - you assign a role for who can manage it (admin) and another role for who can use it
    - this useage-role can become the execution role of lambda - it should have the key usage permissions in its policy
- lambda, ec2, ecs - support hyper-threading
- important port - 22, 80, 433, 