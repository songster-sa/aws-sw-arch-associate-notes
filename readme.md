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
- [](#)
- [](#)
- [](#)
- [](#)
- [](#)
- [](#)
- [VPC](#VPC)
- [Random points](#Random-points)
- [](#)

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
- Spot instances - if aws stops then u r not charged for the partial hr
    - Spot block - 1-6hr to decide
    - Spot Request - one time or persistent
    - Spot fleet - spot instances + on demand instances
- Spot instance strategies
    - capacity optimised
    - lowest price - default
    - diversified
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
- EC2 hibernate 
    - saves RAM to root vol - root vol should be encrypted
    - for on-demand and reserved instances
    - max 60 days
    - RAM <= 150GB
- Placement groups
    - all PGs have to be in same region
    - name has to be unique within ur account
    - cannot merge PGs
    - can move an outside ec2 inside - STOP + CLI or SDK - confirm?
    - types
        - cluster - all in same AZ - high n/w throughputs
        - spread - each on diff hardware - 7 max per AZ - can span across AZ
        - partition - mix - each partition on diff hardware but 1 partition can many many ec2 - can span across AZ

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