# AWS Software Architect Associate Exam Notes 2020

Based on Udemy course by aCloudGuru (Ryan) and Stephane Maarek's course, own research and practise tests.
The topics on which most questions are based are highlighted in bold.

Notes here are incremental to my AWS Developer Associate exam notes :
https://github.com/songster-sa/aws-developer-associate-notes

## Contents
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
- [](#)
- [VPC](#VPC)
- [Random points](#Random-points)
- [](#)

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