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
- 

## 3. Design high-performing architectures


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
