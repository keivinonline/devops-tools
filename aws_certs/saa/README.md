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
## 4. Design secure architectures


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