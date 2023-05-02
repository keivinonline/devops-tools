# aws - solutions architect associate
## 1. Design Resilient Architectures
### Aurora Global Database
- low latency global reads and fast recovery
- primary DB in 1 region and up to 5 secondary DB clusters in different region
### Route 53 health checks
- health checks need to be set to `evaluate target health` to be used in routing policies
- for active-active
    - all records have same A or AAAA records and same routing policy (weighted or latency) 
- For active-passive setup,
1. create a health check for each resource. If for AWS resources, can use alias records and set health check to `evaluate target health`
2. create records for each resource with the same routing policy (weighted or latency) and set the health check for each record
3. create records for secondary resources and specify health check to `evaluate target health` if its in AWS
### Route53 Alias
- alias records can be created at the apex of a domain AKA zone apex in aws
- Alias records allow routing to AWS resources like ELB, S3, Cloudfront etc
### S2S VPN
- site-to-site VPN has max 1.25 Gbps throughput
- to scale this, you could use equal-cost mulit-path (ECMP) which AWS transit gateway supports but not for Virtual private gateway 
- on AWS transit gateway, the following is needed for ECMP
    - 2 or more VPNs
    - Dynamic routing using BGP
### Cloudformation
1. `cloudformation outputs`
- outputs can be used to pass values to other stacks
2. `cloudformation mappings`
- just a key-value pair
3. `cloudformation parameters`
- allow passing custom values to stacks
4. `cloudformation conditions`
- allows you to create resources based on conditions
### Aurora serverless
- on-demand, auto-scaling configuration for Aurora
- auto starts, stops and scales capacity up and down

### EKS - Fargate
- can be used to set up open-source container orchestration like K8s
- can be integrated with other public or private clouds
- can be used with AWS Fargate

### DynamoDB Streams
- ordered flow of information about changes to dynamodb tables
- can be used to capture "before" and "after" images or items

### RDS - parameter group
- parameter groups act as a container for engine configuration values to 1 or more DB
- cannot modify default parameter group
- must create new parameter group and modify config 
    - max_connections

### Route 53 - routing policies
1. weighted
- for blue-green deployments 
- associate multiple resources for a single domain name and specify a weight for each resource
2. multivalue answer 
- route traffic randomly for multiple resources 
### S3 - glacier instant retrieval 
- each object in S3 bucket can have a user-defined storage class
### S3 - multi-part upload
- simple PUT can upload up to 5 GB
- multipart upload can handle 5GB to 5TB

### ECS - data parameter
- used to perform automated config tasks such as 
    - docker daemon config
    - ECS container agent config
### API gateway - integration sources
1. pubic facing HTTP endpoint outside of AWS
2. lambda functions from other accounts
3. VPC link
### API gateway - backend integrations
1. lambda, ec2
2. non-aws hosted HTTP endpoint
### Cloudfront - trigger Lambda@Edge
1. viewer request
2. viewer response
3. origin request
4. origin response
- request
[end_user] -> [viewer request] -> [cloudfront_cache] -> [origin request] -> [server]
- response
[server] -> [origin response] -> [cloudfront_cache] -> [viewer response] -> [end_user]
### Lambda - poll based servies
- kinesis
- dynamoDB
- SQS
### Placement groups
1. Cluster placement group
- puts instances in single AZ 
2. spread placement group
- puts instances across different racks
3. partition placement group
- instances in a partition have their own set of racks
### Privatelink
- private connectivity between VPCs
- need network load balancers to be used in service providers
- network load balancer will perform a source NAT 
### RDS snapshots
- creates a volume snapshot of DB once a day during window
- RDS uploads transaction logs to S3 every 5 minutes
### Shifting AWS Orgs
1. remove all members from old Org
2. delete old org
3. invite management acc of old org to be member of new org
### Database migration service (DMS)
- Schema migration tool can be used to convert sql to nosql for dynamodb
### AWS datasync
- supports lustre file system
### FSx
- FSx for lustre integrates with S3
- FSx can be shared to multiple vpc, regions using inter-vpc, inter-account and region access
- FSx for windows offers singale and multi AZ with options of 
    - SSD and HDD
### ECS -  Dynamic port mapping
- supported by App load balancer and network load balancer
### SNS
- uses pub/sub to provide async event notifications
### Aws batch
- if jobs get stuck in RUNNABLE status, then something is preventing it from being placed on compute resources
1. awslogs log driver not configured 
2. insufficient resources
3. no internet access
4. ec2 instance limit reached

## 2. Design Cost-optimized architectures 
### Redshift Datashare
- datashare is a unit of sharing data that can be created for data sharing across accounts
- to enable datashares, source and destination accounts must be encrypted and in same region
### Resource access manager (RAM)
- share resources
### EFS datasync
- sync data between EFS in different regions
- data does not go through the internet
- can be 1 time migrations or recurring basis
- supports
    - NFS, SMB
    - S3
    - snowcone 
    - FSx
### Private NAT gateway
- provide on-prem or other VPCs from a private subnet connectivity

### EFS - Standard Infrequent Access (IA)
- per GB retrieval fees apply
- can toggle to standard storage class
### AWS Outposts
- on-prem AWS infra to any onprem or edge location
- fully managed solution
### S3 Glacier options
1. instant retrieval
- once per quarter data needs and down to milliseconds
2. flexible retrieval
- once or twice per year data retrieval
- 1-5 mins expedited
- 3-5 hours standard
- 5-12 hours bulk
3. deep archive
- can be restored within 12 hours for standard, 48 for bulk
### Application Migration service
- onprem to AWS for SAP, oracle sql server

### RDS - automated backups
- default retention period of 1 day (0 to 35 days)
- for > 35 days, can copy the snapshot which will be stored as manual snapshot and don't expire
- best practice to export to S3 for Maria, MySQL and Postgresql
### S3 intelligent tiering
- monitors access patterns and moves objects between 2 tiers
- 128KB always remain in S3 standard storage class 
### Redshift cluster - snapshots
- automated snapshots are automatically deleted within max 35 days
### Amazon glacier
1. expedited retrieval
- 1 - 5 mins
2. bulk retrieval
- lowest cost option
- 5-12 hours
3. standard retrieval
- 3-5 hours
### Savings plan
1. compute savings plan
- regardless of instance family, AZ , region and OS
2. ec2 savings plan
- specific instance family
### EBS types
1. Cold HDD
- infrequent usage, not throughout the day
- 125 GB to 16 TB 
### Glacier to S3 move
1. restore objects from glacier deep archive using s3 console
2. use lifecycle policy to move objects to other s3 tier
## 3. Design high-performing architectures
### API Gateway
- `API Caching` - cache backend responses to reduce calls made 
- `Gateway Response` - to customize error responses
- `Method response model` - to set up a response format 
- `Mapping templates`
    - transform request from frontend data formate to backend data format vice versa
    - done via Velocity Template Language (VTL)

### S3 Event notifications
### EventBridge
- start processing after changes to objects store in S3
1. Advanced filtering
- match object parameters for specific notifications
2. Multiple destinations
3. Fast and reliable without custom codes

### Autoscaling Policies
- Default
- AllocationStrategy - rebalancing base on policy
- OldestLaunchTemplate
- OldestLaunchConfiguration
- ClosestToNextInstanceHour - will terminate instance closest to billing cycle
- NewestInstance
- OldestInstance

### Cloudformation - Custom Resources
- write custom provisioning logic in CF template
1. Amazon SNS backed custom resource
- receives notification and invokes logic
2. Lambda-backed custom resource

### EC2 Hibernate
- suspend to disk
- saves contents from RAM to EBS root volume

### Private NAT Gateway
- route traffic between 2 VPCs with overlapping subnets
- create new subnets in each vpc for routing
- create a private NAT in source VPC and an load balancer in dest VPC

### Write intensive workloads for RDS
- place SQS in front of RDS to prevent write operation 
- SQS scales automatically and can be used to buffer writes

### Site to Site vpn setup
- virtual private gateway attached to VPC
- public IP on customer gateway for onprem network
### Security groups
- no inbound traffic is allowed by default
- which is essentially a whitelist
- can't specify deny rules for security groups
### Autoscaling - SQS queues
- SQS queue to store incoming jobs
- custom cloudwatch metric to measure queue size `ApproximateNumberOfMessagesVisible` at 5 mins interval
- target tracking policy that configures autoscaling group based on alarms triggered by custom metric
### Aurora DB trigger Lambda permissions
1. create IAM role for auroraDB
2. grant permissions to invoke lambda function
3. set `aws_default_lambda_role` for DB cluster parameter group
4. associate DB cluster parameter group to DB cluster
5. allow outbound connections from DB to lambda

### EMR and Apache Spark and S3
- Amazon EMR used to process large amounts of data
- supports Apache Spark, HBase, Flink, Hudi ,Hive and presto
- compute and storage are decoupled
- can use S3 as storage 
- clusters can be launched and stopped on demand
### DynamoDB - autoscaling
- adjusts throughput capacity based on workload
- must provide initial settings for read and write 
### Redshift - Kinesis Data Firehose Delivery system
- Redshift as analytical service
1. `Kinesis Data Firehose` as ingestion service
    - need to first store in S3 bucket 
    - then copies data to redshift 
- only stores data up to 24 hours if destination is unavailable
- supports transformation of data
    - compression
    - batch
    - encryption
### Cloud trail - multi region
- create trail and apply to all regions
- logs to specified s3 bucket
### CIDR standards
- aws reserves 5 ip addresses for each subnet
1. 10.0.0.0 network address
2. 10.0.0.1 VPC router
3. 10.0.0.2 DNS service
4. 10.0.0.3 for future use
5. 10.0.0.255 for broadcast
### S3 - performance
- delivers strong read-after-write and list consistency automatically
### EBS storage
1. `throughput optimized HDD (st1)`
- 500 MiB/s 
2. `code HDD (sc1)`
- 250 MiB/s
3. `gp2`
- 250 MiB/s
4. `gp3`
- 1,000 MiB/s
5. `Provisioned IOPS (Block express) io2`
- up to 64 TB anmd 4,000 MiB/s
### DynamoDB - performance
- when each data item < 20KB, use DynamoDB
### S3 - DELETE API call
- when versioning is enabled, DELETE cannot permanently delete an object
- instead, it inserts a delete marker
- returns a 404 if object is current version
- use `delete object versioned` to permanently delete object
### Lambda - when updating versions
- between versions, up to 1 minute window, the requests can be served by either latest of current version of deployment

### Direct Connect + VPN
- can create IPsec-encrypted private connections
### Elastic Fabric Adapter (EFA)
- low latency and high through put for High performance computing
### Auto scaling - lifecycle hooks
- puts instance in wait state before termination
- during this wait state, can perform custom activities 
- default wait state is 1 hour
- a popular hook is to control when instances registers with ELB
###  Global Accelerator
- redirects user requests to nearest edge location and routes data to global network
- reroutes to healthy IPS if fails
- does not rely on IP cached on client devices
- change propagation is in seconds
- gets static IP that provides fixed entry point to applications 
### Cloudformation - change detection
- only checks property values explicitly set by stack templates or template parameters

### Datasync
- can perform opposite direction syncs
- but can't run both direction syncs at the same time
- only support periodically and not simultaneously or multiple active application writes
### DynamoDB streams and lambda
1. associate stream ARN with a lambda function 
2. lambda polls the changes when item is modified or new record created
### redshift and EMR
- redshift performs analytics 
- emr can be used to shift data from kinesis streams to redshift
### Cloudfront
1. pre-signed URLs
- contains user login credentials for specific urls
- allows user to upload specific object to a bucket
- when creating the url, need to provide
    - bucket name
    - object key
    - HTTP method
    - expiration date
### RDS - read replica
- can be created in a different region
### DynamoDB - global tables
- fully managed for multi-region, multi-active database
### Gateway cached volumes
- retains copy of frequent data locally
### Gateway stored volumes 
- for inexpensive data
### Unified cloudwatch agent
1. collect internal system-level metrics for ec2 or on prem
2. collect custom metrics from apps on ec2 using statsD and collectD
3. collect logs from ec2 or on prem 
4. for autoscaling group, can aggregate metrics using "aggregate_dimensions" in agent config

### ECS and outpost
- ec2 launch type with EC2 is supported in outpost for low latency onprem usage
- for consistent high cpu and mem usage
### Redshift - performance data
1. cloudwatch metrics - cpu, latency and throughput of redshift clusters
2. query/load data via redshift console
- custom queries can eb created to query performance data
can support up to 10 Gbps for single-flow traffic
- instances not within cluster placement group can use 5 Gbps for single flow
### DynamoDB - global secondary index
- GSI allows different hash key for the table for complex queries
### ECMP - equal cost multi path
- aws provides 2 vpn endpoints, ECMP can be used to carry traffic on both endpoints to increase performance
### EFA - elastic fabric adapter
- must be be a member of SG that allows all inbound and outbound to itself
- OS-bypass capabilities for non windows
### Codepipeline - S3 source
1. cloudwatch even rule and cloudtrail must be applied
2. disable peeriodic checks
### Placement groups
- can migrate instances between placement groups but merging is not supported
1. cluster placement group
- provides high performance networking
## 4. Design secure architectures

### Network load balancer (NLB) - TLS
- requires 1 certificate per TLS connection
- uses Server name indication (SNI) for selection algorithm for certs
- up to 2048 bits RSA only
### Redshift - encryption at rest
- can modify cluster to enable encryption to use KMS
- cluster will be migrated to a new encrypted cluster and vice versa
###  Cognito
- integrate with facebook, amazon and google etc
1. user pools
- sign in to web or mobile app
2. identity pools
- obtain temp AWS credentials to access AWS services
### Application Load Balancer (ALB) - Certs
- supports SNI (server name indication) which supports multiple domain names with different TLS certs behind single ALB
### AWS Partner Network (APN)
- grant cross account IAM role for third party 
### VPC - extend CIDR range
- 
### Lambda - without need to update code
1. configure lambda functions to use key configurations
- allows encryption of env vars without deploying new code
2. use encryption helpers
- encrypt variables before sending to lambda

### VPC endpoints
1. Gateway endpoints
- can be specified in route table to access S3 and dynamoDB
2. interface endpoints
- extend functionality of gateway endpoints using private IP and route requests to S3 from 
    - vpc
    - onpremise
    - vpc in another region using VPC peering or transit gateway
- does not support dynamodb
### Cloudtrail - integrity validation
- uses sha 256 to hash and RSA for digital signing
- makes it tamper proof
### Lambda - notifications
- use dead letter queue (DLQ) using SQS and SNS for non-processed payloads
### Server migration service
- migrates onprem workloads to EC2 only
### Server migration service connector
- FreeBSD VM installed onprem and works with server migration service
### PHP WAF rule
- Aws managed PHP application rule 
- add in web ACL of WAF
### Cloudfront
1. pre-signed URLs
- contains user login credentials for specific urls
- does not ensure contents can only be accessed through cloudfront
- for individual files only
2. Restrict access to cloudfront only
- Create Origin Access Identity (OAI) in cloudfront
- use S3 bucket policy to only allow OAI to access
3. Origin with cookies
- use cookies to restrict access to cloudfront
- access to multiple files
### RDS - enable encryption on existing db
1. create snapshot of unencrypted DB
2. copy snapshot and enable encryption
3. restore the DB
### Global accelerator, ELB, IoT
- provides 2 anycast static ip addresses
### Athena
- decrypts s3 data automatically
### AWS org- cloudwatch events
- can use cloudwatch events to raise actions for aws org
### Cloudfront -> s3
1. Origin access identity (OAI)
- legacy method that does not support KMS or dynamic requests to s3
2. origin access control (OAC)
- new method
- principal element should be "cloudfront.amazonaws.com" and the condition should match cloudfront distribution which contains s3
### ECS networking
1. host mode
- basic mode where container network is tied to host
2. bridge mode
- network between container and host
- allows remapping of ports between host and container ports
3. non
- no external connection
4. awsvpc mode
- each task has a separate ENI with separate IP and SG 
- separate security policies for each task
### S3 and dynamodb endpoints
1. Gateway endpoints
- establish from private subnet  -> s3 or dynamodb
- flows through private links
2. service endpoint
- can be accessed from onprem 

### Cloud HSM - backup to s3
1. generates unique ephemeral backup key (EBK) to encrypt data using aes 256
2. the EBK is then encrypted using persistent backup key (PBK) using AES 256 as well in the same region
- 
## Services recap
### Rekognition
- identify objects and etc in images or videos
- also does facial recognition and facial search capabilities
### Textract
- managed ML service to extract text from images and documents
### Comprehend
- NLP service to uncover connections in text
- mainly used for sentiment analysis
### Polly
- turns text to speech
### Cloudtrail
- track and records API calls
- can't tract requests on application depth
### Inspector
- discover and scan vulnerabilities and unintended network exposure
- automated security assessments
### Cloudwatch
- collect logs
### X-ray
- analyze and debug distributed applications especially microservices
- integrates well with SQS
### Private link
- underlying for VPC interface endpoints

### Kinesis Data Firehose
- capture, transform and load various sources to 
    - S3
    - Redshift
    - Data lakes etc
- does not capture data from video streaming devices
### Kinesis Data Analytics
- gate streaming data from Kinesis Data Streams and S3 and IoT
- creates alerts or respond in real-time
### Kinesis Video Streams
- for video playback, face detection, security monitoring etc
- can use Rekognition after this
### Kinesis Data Streams
- capture GBs of real-time data 
- does not do video streams 

### Aurora Parallel Query
- only for MySQL
- perform queries over data stored in db without copying data to separate system

### Redshift Spectrum
- query data directly from files in S3 
### Redshift Advanced Query Accelerator (AQUA)
- speeds up queries by caching results in memory

### Cloudformation
1. cfn-init
- retrieve and interpret metadata from AWS::Cloudformation::Init
- installs packages, configures services, creates files and etc
2. cfn-hup
- checks updates to metadata and executes custom hooks
3. cfn-signal
- indicate software or app is updated on EC2
4. cfn-get-metadata
- retrieve metadata for a resource via a key 

### Application Discovery Service
- collects data from onpem
1. agentless discovery
- suited for VMware 
- deploys an OVA 
2. agent-based discovery
- best suited for physical servers

### Snowball Edge
- has different flavours
1. data transfer 
- supports 100 TB (80TB usage) 
2. ec2 compute
- 40 vCPU, 80 GB mem and 1 TB ssd
3. compute optimized
4. compute optimized with GPU

### Snowcone
- only supports 8TB HDD 
### AWS transfer family with FTPS
- for SFTP, FTPS and FTP to and from S3 


### WAF
1. blanket rate-based rule
- block source IP addresses that generate excessive requests
2. URI-specific rate-based rule
- blocks IP making requests to specific URI
- useful to prevent DDOS attacks and expensive resources
3. IP reputation rate-based rule
- prevents malicious IPs from making requests to app
4. managed group rules
- would not track rate of requests

### AWS Orgs - SCP
- SCPs do not have impact on users and roles in management accounts
- can use "S3 block public access" to block public access to S3 buckets in all accounts

### AWS EKS Anywhere
- can be used to deploy K8s infra at onprem
- EKS anywhere uses VMware vSphere for this deployment
- Amazon Distro can be used for distribution of K8s and dependencies deployed by EKS to K8s clusters on prem
- EKS anywhere doe not work with Outposts

## AWS S3 - Glacier Vault
- can be attached with 
    - 1 vault access policy 
    - 1 vault lock policy
- Access policy - who can access the vault
- Vault lock policy - no changes can be done once locked

### AWS SSO (previously named Identity Center)
- has built-in support for other applications like Salesforce, Jenkins etc
- has flexibility of having user database in IAM, AWS managed AD or on
- integrates with Identity provider that is SAML 2.0 compliant

### CloudTrail Lake
- fully managed service to collect, store and analyze CloudTrail logs

### GuardDuty
- fully managed advanced threat detection service 
- identify threats like account, instance nad bucket compromise
- process data that can be used to generate thread detections only, hence saving costs

### Macie
- protects sensitive data from unauthorized access in S3

### Systems Manager
- manage and remediate operational issues

### AWS Inspector - Run Vulnerability Assessments
- define assessment target and include all instances in AWS and region
- 
### AWS KMS
1. Asymmetric KMS Keys
- private key never leaves KMS unencrypted
- used for digitally signing and verifying messages, transactions, tokens etc
2. Symmetric KMS Keys
- default encryption method for KMS
- same key for encryption and decryption
- 

### AWS RDS - TDE
- Transparent Data Encryption (TDE) is supported for Oracle and SQL servers only
- TDE with Oracle can be integrated with AWS CloudHSM which stores, generates and manages kes in the hardware security module.

### AWS Shield
- provides higher protection to DDoS attacks
- provides real-time visibility of attaches and can be integrated with WAF 
- AWS shield is automatically enabled for AWS resources 
- AWS Shield Advanced
    - Shield Response Team (SRT) is engaged with customer
    - needs to be enabled on other resources

### VPC endpoints
1. accessing KMS keys
- restricted via matching VPC Endpoint ID or VPC ID 
`aws:sourceVpce` - restricts access based on VPC endpoint
`aws:sourcevpc`  based on VPC that hosts private endpoint