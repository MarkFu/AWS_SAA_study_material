[toc]


# To-do
- read [exam readiness](https://aws.amazon.com/training/course-descriptions/exam-workshop-solutions-architect-associate/)
- read AWS FAQ
- Jayendra's blog
- practice exams
	- A Cloud Guru (2) - *purchased*
	- John Bonso on Udemy (6) - *purchased*
	- Whizlab (21) - *to purchase*
	- official practice exam (20 questions) for $20? - *to purchase*

# To practice
- create your own blog (e.g. WordPress)
- create your own VPN
- create your own scraping server
	- with database backend and dashboard in frontend

# Topic notes

## Management

### IAM
- key entities
	- users
	- groups
	- roles
	- policies (in JSON format)
		- identity policy
		- resource policy
- universal: not specific to region
- new users have no permissions when first created
- Access key ID and secret access keys are assigned to new users
	- not same as passwords; can only be used via APIs and command line
	- can only be viewed once
	- (for EC2) better to use roles instead of keeping credentials
- you can give federated users single sign-on (SSO) access to AWS management console with SAML (Security Assertion Markup Language)
- Amazon Resource Name (ARN): uniquely identifies any AWS resource
	- begins with `arn:partition:service:region:account_id:`; ends with `resource` or `resource_type`
	- e.g. `arn:aws:ec2:us-east-1:1234523121:instance/*`
- IAM policy has no effect until attached
- IAM policies rules
	- not explicitly allowed means implicitly denied
	- explicit deny > everything else
	- AWS joins all applicable policies
	- AWS-managed vs. customer-managed
- in-line policy: only applicable to specific roles
- permission boundaries
	- used to delegate admin to other users
	- prevent privilege escalation or unnecessarily broad permissions
	- control maximum permissions an IAM policy can grant

### AWS Organization
- paying account should be used for billing purpose only; do not deploy resources in paying account
- enable/disable AWS services using Service Control Polices (SCP) either on OU (Organization Unit) or or individual accounts
- RAM: Resource Access Manager
	- can share AWS resources between accounts
	- e.g. EC2, Aurora, Route 53, resourece groups
- SSO helps centrally manage access to AWS accounts
	- exam tip: if you see SAML in question, look for SSO in answers


### AWS Directory Service
- a family of managed services heavily integrated with Microsoft Active Directory (AD)
- connect AWS resources with on-premise AD
- standalone directory in the cloud
- use existing corporate credentials
- enable SSO to any domain-joined EC2 instance
- provides AD domain controllers (DCs) running Windows Servers
	- reachable by applications in VPC
	- extend existing AD to on-premises using AD Trust
- Simple AD: standalone managed directory
	- basic AD features
	- easer to manage EC2
	- does not support trusts
- AD Connector
	- directory gateway for on-premises AD
	- avoid caching information in the cloud
	- allow on-premise users to log in to AWS using AD
	- join EC2 instances to existing AD domain
	- scale across multiple AD connectors
- Cloud Directory
	- directory-based store for developers
	- use cases: org charts
	- fully managed service
- Cognito User Pools
	- managed user directory for SaaS applications
	- sign-up and sign-in for web or mobile
	- works with social media identities
- compatibility
![AD Compatibility](pic/AD_compatibility.png)


### Cognito: Web Identity Federation
- **Web Identity Federation** lets you give users access to AWS resources after they are authenticated with a web-based identity provider like Google
	- after authentication, user gets an authorization code from the web ID provider, which can be traded for temporary AWS credentials
- Cognito provides Web Identity Federation with the following features
	- sign-up and sign-in to your apps
	- access for guest users
	- acts as an Identity Broker between your application and web ID provider; no need to write additional code
	- synchronize user data for multiple devices
	- recommended for all mobile AWS services
- Cognito brokers between the app and Google/Facebook to provide temporary credentials which map to an IAM role allowing access to the required resources
- no need for the application to embed or store AWS credentials locally on the device
- **user pools** are user directories used to manage sign-up and sign-in functionality
	- users can sign in directly to the User Pool or using Facebook/Google etc.
	- Cognito acts as an identity broker between the identity provider and AWS
	- successful authentication generates a JSON Web token (JWT)
- **identity pools** enabled to provide temporary AWS credentials to access AWS services like S3 or DynamoDB
	- identity pools are about authorizing access to AWS resources
	- user pools are about users (e.g. email address, passwords...)
![pool](pic/Cognito_pool.png)
- track the association between user identity and the various different devices they sign in from
	- uses Push Synchronization to push updates and synchronize user data across multiple device
	- uses SNS to send a notification to all the devices associated with a given user identity whenever data stored in the cloud changes


### CloudWatch
- a monitoring service for AWS resources and applications run on AWS
- CloudTrail: a record of your management console activities and API calls (a log of who did what at when)
- CloudWatch is for monitoring performance; CloudTrail is for auditing
- standard monitoring with EC2 is 5 minutes; detailed monitoring is 1 minute
- CloudWatch Events can monitor state changes
- CloudWatch can be used to create dashboards, set alarms, monitor events, use logs


### Auto Scaling
- groups
	- logical component: webserver group, application group, or database group
- configuration templates
	- a launch template or a launch configuration for its EC2 instances
	- basically an instruction for what instances to launch, what size they are and etc.
	- to specify information such as AMI ID, instance type, key pair, security groups
- scaling options
	- ways to scale groups
	- can scaled based on occurrence of a specified condition (dynamic scaling) - such as CPU utilization - or on a schedule
	- 5 options:
		- maintain current instance levels at all times
		- scale manually
		- scale based on a schedule
		- scale based on demand
		- use predictive scaling

### General Security Risks
- bad actors: typically automated process
	- content scrapers
	- bad bots
		- fake user agent
		- Denial of Service (DoS)
- in addition to NACL where you can specify (a range of) IP address to block, you can also use host-based firewall
	- NACL operations on layer 4
- when using an Application Load Balancer, the connection from bad actor terminates at ALB, not at the EC2 instance behind
	- a host-based firewall will be ineffective in this case; still need a NACL
	- can set up a WAF before ALB
		- operates on layer 7
		- works best for SQL injection attacks, cross-site scripting attacks
		- may have a configuration of CloudFront
![WAF_CloudFront](pic/WAF_CloudFront.png)
- when using a Network Load Balancer, traffic goes to EC2 instance -> only counter-measure is to use NACL

### KMS: Key Management Service
- **regional** secure key management for encryption and decryption
	- if you have an encrypted resource in one region and you want to move it to another region, must first decrypt, then move, then encrypt in new region
- manages **customer master keys (CMKs)**
	- each associated with a key policy
	- rotated periodically
	- 3 types of CMKs
	![CMK](pic/CMKs.png)
	- CMK symmetry
	![CMK sym](pic/CMK_sym.png)
- ideal for S3 objects, database passwords and API keys
- encrypt and decrypt data up to 4kB in size
	- can use Data Encryption Key (DEK) for larger size
- integrated with most AWS services
- pay per API call
- has audit capability using CloudTrail
- **FIPS 140-2 Level 2**
	- just need to show proof of tampering
	- CloudHSM is level 3 (more stringent)

### CloudHSM: Hardware Security Module
- **dedicated** HSM
- **FIPS 140-2 Level 3**
- manage your own keys
- no access to the AWS-managed component
- runs within a VPC in your account
- single tenant, dedicated hardware, multi-AZ cluster
	- not highly available by default; need to provision HSMs across AZs
- industry-standard: no AWS APIs
- good for softwares with compliance requirement - e.g.
	- PKCS#11
	- Java Cryptography Extensions (JCE)
	- Microsoft CryptoNG (CNG)
- irretrivable if lost

### Parameter Store
- component of AWS Systems Manager (SSM)
- secure serverless storage for configuration and secrets
	- passwords
	- database connection strings
	- license codes
	- API keys
- values can be stored encrpyted (KMS) or plain text
- separate data from source control
- can store parameters in *hierachies*
- can track versions
- can set TTL to expire values such as passwords
- can be integrated with CloudFormation


## Storage

### S3: Simple Storage Service
- S3 is **Object-based**
	- key (name of the object)
	- value (data)
	- version ID
	- metadata
	- sub-resources
		- access control lists (ACL)
		- torrent
- Files can be between 0-5TB
- unlimited storage space
- cannot install operating system on
- Files are stored in **Buckets**
	- up to 100 buckets per account by default
	- by default, all new buckets are private
	- buckets can be configured to create access logs which log all requests made to the S3 bucket
- S3 is a **universal** namespace (globally)
	- URL styles
		- virtual style: ```https://<bucketname>-s3-<region>.amazonaws.com/<filename>```
		- path style: ```https://s3-<region>.amazonaws.com/<bucketname>```
		- static hosting: ```https://<bucketname>-s3-website-<region>```
		- legacy global endpoint (limited support and discouraged)
- **HTTP 200 code** returned to browser if upload to S3 is successful
- consistency model
	- **read after write consistency for PUTS of new Objects**
	- **eventual consistency for overwrite PUTS and DELETES**
- standard availability
	- **99.99%** availability by Amazon (chance of the file being available)
	- **11x9s** durability (chance of not losing the file)
- S3 features
	- **tiered storage**
		- **standard**
		- **IA** (infrequent accessed): less frequent but rapid access, low storage fee but retrieval fee charged
		- **One Zone-IA** (no multiple AZ): similar to Reduced Redundancy; good to store data that is easy to reproduce; 99.50% availability
		- **Intelligent tiering** (by ML): standard + IA
		- **Glacier**: retrieval time can be configured
		- **Glacier Deep Archive**: retrieval time about 12 hours
	- **lifecycle** management
		- automates moving the objects between different storage tiers
		- integrates with versioning; can be on current or previous versions
	- versioning
		- once enabled, cannot be disabled, only suspended
		- integrates with lifecycle rules
		- has MFA Delete capability: has to provide MFA auth to delete a file
	- **cross region replication**
		- must have versioning enabled on both source and destination buckets
		- when turned on, files in bucket before turning on are not automatically replicated
		- delete markers and deletions are not replicated cross region
	- encryption
		- encryption in transit is achieved by SSL/TLS
		- encryption at rest (server side) is achieved by
			- S3 managed keys: SSE-S3 (AES)
			- AWS key management service: SSE-KMS (has quota for number of requests; upload and download count towards quota)
			- Customer provided keys: SSE-C
		- encryption at rest at client side
	- MFA Delete
	- S3 object lock
		- write once, read many (**WORM**) model
		- can be on single or multiple objects
		- governance (can't overwrite or delete without permission) vs. compliance mode (can't be modified by anyone, including root user)
		- legal hold (no retention period)
	- S3 Glacier vault lock
		- with vault lock policy: once locked, policy cannot be changed
	- performance enhancement
		- **prefix**: increase performance by spreading reads across prefix
			- more prefixes, more requests you can do at the same time
		- **multipart uploads** (required for >5G file)
		- S3 byte-range fetches: download large files by parts
		- S3 Select: retrieve partial data of an object by SQL
			- Glacier Select
	- share S3 buckets across accounts (3 ways)
		- Bucket policy & IAM (bucket level; programmatic only)
		- Bucket ACL & IAM (object level; programmatic only)
		- Cross-account IAM roles (programmatic and console)
	- transfer acceleration through CloudFront
	- **DataSync**
		- moves large amounts of data
		- used with NFS- and SMB-compatible file systems
		- replication can be done hourly, daily, or weekly
		- install the DataSync agent to start the replication
		- can be used to replicate EFS to EFS
- S3 cost
	- storage (depending on tiers)
	- (number of) requests and data retrieval
	- storage management pricing
	- data transfer
	- transfer acceleration (use CloudFront)
	- cross region replication
- Athena
	- interactive query service to query data in S3 with SQL
	- serverless
	- can be used to query logs, generate business reports and etc.
- Macie
	- ML and NLP solution to identify and protect sensitive data stored in S3
		- e.g. Personally Identifiable Information (PII)
	- can be used to analyze logs

### Snowball
- Snowball
	- petabyte-scale data transport solution
	- 50TB or 80TB
	- can import to and export from S3
- Snowball Edge: 100TB device with on-board storage and compute capabilities
	- e.g. carried in aircrafts for testing
	- portable AWS (without having to access cloud/internet)
- Snowmobile
	- Exabyte-scale data transport solution; essentially a truck

### Storage Gateway
- connects on-premise software appliance with cloud-based storage
- a virtual or physical device to replicate your on-prem data on AWS
- 3 types of Storage Gateway
	- File Gateway (for flat files; stored directly on S3): network file system (NFS) & SMB
	- Volume Gateway (iSCSI)
		- stored volume: entire local data stored on site and asynchronously backed up to S3
		- cached volume: entire data stored on S3 and frequently accessed data cached on site
	- Tape Gateway (VTL: virtual tape library)

### EFS: Elastic File System
- file storage system for EC2
	- Linux only; does not support Windows instance
- can be **shared** across different EC2 instances (unlike EBS)
  - Amazon EFS supports one to thousands of Amazon EC2 instances connecting to a file system concurrently
- grow and shrink automatically (unlike EBS); only pay for the storage you use
- useful for distributed, highly resilient storage
	- can support thousands of concurrent NFS connections
- supports **Network File System version 4 (NFSv4) protocol**
	- one of the first network file sharing protocols native to Unix and Linux
- data is stored across multiple AZ's within a region
- read after write consistency

### FSx
- Windows FSx
	- Windows Server that runs Windows SMB-based (Server Message Block) file services
	- designed for Windows and Windows applications
	- support AD users, access control lists, groups and security policies, along with Distributed File System (DFS) namespaces and replications
- Lustre FSx
	- file system optimized for compute-intensive workloads, such as HPC, ML,media data processing workflows, electronic data automation (EDA)
	- can store data directly on S3



## Compute

### EC2: Elastic Compute Cloud
- a web service that provides resizable compute capacity
- pricing model
	- **On demand**
	- **Reserved** (1yr or 3yr contract terms)
		- standard reserved
		- convertible reserved (can change instance types)
		- scheduled reserved instances
	- **Spot**: bid price for instance capacity
		- if Spot instance is terminated by AWS, no charge for partial hour of usage; 
		- if you terminate the instance yourself, you will be charged for any hour in which the instance ran
		- decide max spot price; the instance will be provisioned so long as price is lower
		- can use **Spot block** to allow >max price for 1-6 hours
		- not good for persistent workload, critical jobs, and databases
		- **Spot Fleets**: a collection of Spot instances (and optionally on-demand instances)
	- **Dedicated host**
- AMI (Amazon Machine Image) is simply a packaged-up environment that includes all the necessary bits to set up and boot your instance
  - Your AMIs are your unit of deployment
- AMI can be selected/configured based on
	- region
	- operating system
	- architecture (32 vs. 64 bit)
	- launch permissions
	- storage for root device
		- **instance store** (ephemeral storage): template stored in S3
			- can scale to millions of IOPS
			- can only be terminated or rebooted; cannot be stopped. If underlying host fails, you will lose your data
			- once stopped, data will be lost
			- cannot add instance store volume after provisioned; can still add EBS volume
			- cannot be seen under EBS Volume in the Console (because it's not EBS)
			- root volume will be deleted on termination. no option to keep root device (unlike EBS volumes)
			- an inexpensive way to launch instances where data is not stored to the root device
		- **EBS backed volumes**
			- scale up to 64000 IOPS
			- By using Amazon EBS, data on the root device will persist independently from the lifetime of the instance.
- **instance types**
	- Accelerated Computing instance family is a family of instances which use hardware accelerators, or co-processors, to perform some functions, such as floating-point number calculation and graphics processing, more efficiently than is possible in software running on CPUs
- security groups
	- specify inbound and outbound rules
	- rule change takes effect immediately
	- security groups are **stateful**: if you open a port, it will be open for both inbound and outbound
		- unlike ACL which is stateless
	- if you create inbound rule for HTTP, same outbound rule is automatically created
	- all inbound traffic is blocked by default; all outbound traffic is allowed
	- cannot block particular port, type or IP address; can specify allow rule but not deny rules
	- can have multiple security groups attached to one EC2
	- can have multiple EC2 instances with the same security group
	- security group can talk to each other: e.g. set up inbound rule to allow traffic from another security group
- network options
	- **ENI** (Elastic Network Interface)
		- essentially a virtual network card
		- allows IPv4 addresses from the range of your VPC; with security groups, MAC address and etc.
		- use case: basic networking; create a management network; use network and security appliances in your VPC; create dual-homed instances
	- EN (Enhanced Networking)
		- use single root I/O virtualization to provide high-performance networking capabilities; provides higher bandwidth, higher packet per second (PPS)
		- use case: when you want good network performance (speed up to between 10-100Gbps)
		- can be enabled via Elastic Network Adapter (**ENA**) for up to 100Gbps or Virtual Function (VF) for up to 10 Gbps (usually go with ENA)
	- **EFA** (Elastic Fabric Adapter)
		- a network device you can attach to EC2 instance to accelerate High Performance Computing (HPC) and ML applications
		- can use OS-bypass: enable HPC and ML applications to bypass operating system kernel and to communicate directly with EFA device. Linux only
		- use case: HPC and ML applications; OS-bypass
- encryption (of root device)
	- EBS root volumes of your default AMI's can be encrypted (unlike before); additional volumes can be encrypted
	- to encrypt a volume after provisioned, create a snapshot, make a copy, and encrypt the copied snapshot, create AMI based on it, launch instance from AMI
	- snapshots must be unencrypted to be shared
- hibernation
	- saves the contents from the instance memory (RAM) to EBS root volume
		- instance RAM must be less than 150GB
	- operating system performs hibernation (suspend-to-disk); not rebooted
	- instance boots much faster. good for long-running processes and services that take time to initialize
	- previously attained data volumes are reattached; instance ID remains the same (unlike stop-restart)
	- needs to enable hibernation when provisioning the instance; root volume must be encrypted
	- can't be hibernated for more than 60 days
	- available for on-demand and reserved instances
- **placement group**
	- **clustered**
		- grouping of instances within a single AZ (cannot span multiple AZ's)
		- low network latency, high network throughput
		- recommend to have homogeneous instances within clustered placement groups
	- **spread**
		- a group of instances that are each placed on distinct underlying hardware
		- recommended for applications that have a small number of critical instances that should be kept separate from each other
		- up to 7 running instances per AZ
	- **partitioned**
		- each partition has its own rack, network and power sources
		- each partition can have multiple instances, but each partition is separate from each other
	- other features
		- unique placement group name within AWS account
		- cannot merge placement groups
		- only certain types of instances can be launched in a placement group (Compute Optimized, GPU, Memory Optimized, Storage Optimized)
		- existing instance can be moved to a placement group; needs to be stopped first; cannot be moved via console yet
- metadata
	- information about the instance itself (e.g. public IPv4 address)
	- can be retrieved via a special URL (```http://169.254.169.254```) or using the API via CLI or an SDK
	- can assign your own metadata in the form of tags

### EBS: Elastic Block Store
- provides persistent block storage volumes for use with EC2 instances (essentially virtual hard disk drive)
- automatically replicated within AZ
- termination protection is turned off by default
- on an EBS-backed instance, the default action is for the root EBS volume to be deleted when the instance is terminated
	- additional volumes will remain (by default)
- types
  ![EBS types](pic/EBS_TYPES.png)
  - SSD-backed storage for **transactional** workloads (performance depends primarily on IOPS) and HDD-backed storage for **throughput** workloads (performance depends primarily on throughput, measured in MB/s)
    - SSD-backed volumes are designed for transactional, IOPS-intensive database workloads, boot volumes, and workloads that require high IOPS
    - HDD-backed volumes are designed for throughput-intensive and big-data workloads, large I/O sizes, and sequential I/O patterns
- EBS volume needs to be in the same AZ as EC2
- EBS volume can be attached to multiple EC2 instances
- volume can be changed (type and size) after EC2 is provisioned; may take some time to take place
- to move to different AZ, create a snapshot; then create an image (AMI) based on the snapshot (with hardware-assisted virtualization); launch EC2 from AMI in another AZ or copy AMI to another region
- additional EBS volume (not root device) can be detached without stopping the instance
- snapshots exist on S3
- EBS snapshots are only available through the Amazon EC2 APIs (not S3 APIs)
- snapshots are incremental; only the blocks that have changed are captured in latest snapshots
- best to take snapshots when instance is stopped (can still take snapshot if running)


### Lambda
- takes care of provisioning and managing the servers to run your code
- use cases
	- event-driven compute service where Lambda runs your code in response to events (e.g. data change in S3 bucket)
	- run code in response to HTTP requests using API Gateway or API calls
- use Lambda to directly interact with backend database (and skip EC2 instances where you need to manage all the configurations)
![serverless](pic/serverless.png)
- continuously **scaling out** (automatically)
	- **each event triggers an individual Lambda function**
- **Lambda functions can trigger Lambda functions**
- priced based on:
	- number of requests: first 1 million requests are free; $0.2 per 1 million requests thereafter
	- duration: rounded up to 100ms; also depends on memory allocated - e.g. $0.00001667 per GB-second
- AWS X-ray allows you to debug serverless applications (as architecture can get complex)
- Lambda can do things **globally** (e.g. back up S3 buckets to another S3 bucket)
- possible triggers
	- API Gateway
	- CloudWatch events
	- S3
	- DynamoDB
	- Kinesis
	- SNS
	- SQS
	- ...
- RDS cannot trigger Lambda
- Lambda supports hyper-threading on one or more virtual CPUs


## Network

### CloudFront
- content delivery network (CDN)
- **edge location**: the location where content is cached
	- not read only; can write to edge locations too
	- objects are cached for the life of the TTL (time to live)
	- you can clear cached objects (invalidation), but you will be charged
- origin: origin of all the files to be distributed. Can be S3, EC2 instance, Elastic Load Balancer, or Route53...
- distribution: a collection of edge locations given to CDN
- web distribution vs. RTMP (for media streaming)
- Restricted access
	- CloudFront signed URLs: 1 file for 1 URLs
	- CloudFront signed cookie: 1 cookie for multiple files
	- policy attached when signed URL or cookie is created
		- URL expiration
		- IP range
		- trusted signers
	- use CloudFront signed URL/cookie for EC2 and etc. 
		- S3 signed URL is an alternative for S3 only
	- S3 signed URL
		- issues a request as the IAM user who creates the pre-signed URL
		- limited lifetime
		- only if the user can access S3 directly (not usually the case)

### WAF: Web Application Firewall
- monitor HTTP and HTTPS requests
- layer-7 aware firewall: specific to application
- 3 types of behaviors:
	- allow all requests except the ones you specify
	- block all requests except the ones you specify
	- count the requests that match the properties you specify
- protection against web attacks with the following possible conditions:
	- IP address
	- country
	- values in request headers
	- strings that appear in requests
	- length of requests
	- malicious SQL code (SQL injection)
	- malicious script (cross-site scripting)

### Route 53: DNS
- convert human friendly domain names to Internet Protocol (IP) address
- IPv4 (32 bits) addresses are running out, hence IPv6 (128 bits)
- top level (.com) and second level domain (.uk)
- top domain names are controlled by by the Internet Assigned Numbers Authority (IANA) in a root zone database
	- IANA [database](http://www.iana.org/domains/root/db)
- domain registrar: an authority to assign domain names
	- e.g. Amazon, GoDaddy.com
	- may take 3 days to register (with Amazon)
- Start of Authority (SoA): every DNS begins with this
	- name of the server that supplied the data for the zone
	- administrator of the zone
	- current version of the data file
	- default number of seconds for the time-to-live file on resource records (usually 48 hours)
- **Name Server (NS)** records: used by top level domain servers to direct traffic to the content DNS server
- **A Record** (A=Address): translates domain name to numerical IP address
	- e.g. `http://www.acloud.guru` to `http://123.10.10.80`
- **CNAME** (Canonical Name): resolve one domain name to another
	- e.g. `m.acloud.guru` and `mobile.acloud.guru` mapped to same IP
	- cannot be used for naked domain names (e.g. cannot work for `http://acloud.guru`)
- **Alias Records**: map resource record sets to load balancers, CloudFront distributions or S3 (configured as websites)
	- like CNAME
	- always choose Alias Record over CNAME
- Elastic Load Balancer (ELB) does not have pre-defined IPv4 addresses; resolve to them using a DNS name
- **Routing Policy**
	- simple routing
		- one record with multiple IP addresses
		- if multiple values specified in record, Route 53 returns all values in a random order
	- weighted routing
		- split traffic based on weights assigned
	- latency-based routing
		- route traffic based on lowest network latency for end user
	- failover routing
		- for active/passive (primary/secondary) setup
		- if active record set fails health check, traffic is directed to passive record set
	- geolocation routing
		- send traffic based on geographic location of the end user
	- geoproximity routing (traffic flow only)
		- route traffic based on geographic location of user and resource
		- can also use bias to assign traffic -> expands or shrinks the size of a geographic region
		- must use Route 53 traffic flow
	- multivalue answer routing
		- like simple routing and allows to put health check on each record set
- can set **health checks** on individual record sets; if fails, will be removed from Route 53 until it passes the check

### VPC: Virtual Private Cloud
- provision a logically isolated section of AWS cloud
- consists of IGWs (or Virtual Private Gateways), Route Tables, Network Access Control Lists, Subnets, and Security Groups
![VPC](pic/VPC_pic.png)
- 1 subnet = 1 Availability Zone
- Security Groups are *stateful*; Network ACLs are *stateless*
- NO transitive peering between VPCs
- only 1 Internet Gateway (IGW) per VPC
- security groups cannot span VPCs (a security group does not show up in another)
- when creating a VPC, **subnet** and **IGW** are NOT created automatically; route table, NACL and security group are created by default
- US-East-1A AZ in my AWS account is not the same as in someone else's AWS account - all randomized
- Amazon reserves 5 IP addresses within your subnets
- need a minimum of 2 public subnets to deploy an internet facing (application) load balancer
- by default, any user-created VPC subnet will not automatically assign IPv4 address to instances (except for the "default" VPC created by AWS) 
	- easiest solution is to create an Elastic IP and associate it with your instance
- Some vulnerability scans are allowed without alerting AWS; some require you to alert AWS
- by default, instances in new subnets in a custom VPC can communicate with each other across AZ
- when you create a new security group, all outbound traffic is allowed by default
- can create up to 5 VPCs in each AWS region
- in VPC, an instance does not retain its private IP
- **Network Address Translation (NAT)**: ways to let private subnets to connect to Internet
![NAT](pic/NAT.png)
	- **NAT instance**: EC2 instance
		- need to disable source/destination checks
		- must be in public subnet
		- must be a route out of the private subnet to the NAT instance
		- always behind a security group
		- can easily become a bottleneck; instance size dictates the upper limit of traffic 
		- can create high availability using auto-scaling groups, multiple subnets in different AZs, and a script to automate failover
		- gradually phasing out
	- **NAT gateway**: highly available gateway
		- redundant inside the AZ
		- starts at 5Gbps and scales to up to 45Gbps automatically
		- not associated with security Group
		- no need to patch
		- automatically assigned a public IP
		- 1 NAT gateway per AZ
			- if multiple AZs share one NAT gateway and if it's AZ fails, resources in other AZs lose Internet access
			- good practice to create 1 NAT gateway in each AZ
- Network ACL: Network **Access Control List**
	- default network ACL allows all outbound and inbound traffic
	- any custom network ACL denies all inbound and outbound traffic by default
	- each subnet in the VPC must be associated with a network ACL; if not specified, then default ACL
	- block IP address with network ACLs, not Security Groups
	- can associate a network ACL with multiple subnets
	  - a subnet can only be associated with one ACL at a time
	- ACL contains a numbered list of rules; rules are evaluated in numerical order
	  - if you want to deny something, always put it before the allow rule
	- always have separate inbound and outbound rules; each rule can allow or deny traffic
	- *stateless*: responses to allowed inbound traffic are subject to the rules for outbound traffic (and vice versa)
	- act first, before security groups
- VPC **Flow Logs**
	- capture information about IP traffic in/out of network interfaces
	- stored using CloudWatch logs
		- need to have a CloudWatch log group in place
		- can also send to S3
	- 3 levels
		- VPC level
		- subnet level
		- Network Interface level
	- flow logs for peered VPC must be in the same account
	- can tag flow logs
	- after creation, cannot change flow log configuration (e.g. cannot change IAM role)
	- not all IP traffic is monitored, e.g.
		- traffic generated by instances when they contact Amazon DNS server (unless you use your own DNS server)
		- traffic generated by Windows instance for Amazon Windows license activation
		- traffic to and from `169.254.169.254` for instance metadata
		- DHCP traffic
		- traffic to the reserved IP address for the default VPC router
- **Bastion** Host
	- a special purpose computer on a network specifically designed and configured to withstand attacks
	- generally hosts a single application (e.g. proxy server)
	- a NAT Gateway or NAT instance is used to provide Internet traffic to EC2 instances in a private subnet
	- a bastion is used to securely administer EC2 instances (using SSH or RDP)
	- cannot use a NAT Gateway as a Bastion host
![bastion](pic/bastion.png)
- VPC **endpoints**
	- enables you to privately connect your VPC to supported AWS services powered by Private Link
	- Instances in VPC do not require public IP addresses to communicate with resources in the service
	- traffic between VPC and the other service **does not leave the Amazon network**
	- endpoints are virtual devices that are horizontally scaled, redundant and highly available VPC components
	- two types of endpoints
		- **interface endpoints**
			- an elastic network interface (ENI) with a private IP address that serves as an entry point for traffic destined to a supported service
		- **gateway endpoints**
			- like a NAT gateway
			- support S3 and DynamoDB
![endpoint](pic/endpoint.png)
- **Private Link**
	- best way to expose a service VPC to many (e.g. thousands of) customer VPCs
	- does not require VPC peering; no route tables, NAT, IGWs, etc.
	- requires a **Network Load Balancer** on the service VPC and an **ENI** on the customer VPC
![privatelink](pic/private_link.png)
- **transit gateway**
	- a way to simplify network topology
	- allows transitive peering between many VPCs and data centers
	- works on a hub-and-spoke model
	- works on a regional basis; can have it across multiple regions
	- can use it across multiple AWS accounts using RAM (Resource Access Manager)
	- can use route tables to limit how VPCs talk to one another
	- works with Direct Connect as well as VPN connections
- supports *IP multicast* (which is not supported by other AWS service)
	![transit](pic/transit_gateway.png)
- VPN CloudHub
	- if you have multiple sites, each with its own VPN connection, you can use AWS VPN CloudHub to connect those sites together
	- hub-and-spoke model
	- low cost; easy to manage
	- operates over public network, but all traffic between the customer gateway and the AWS VPN CloudHub is encrypted

### Direct Connect
- a cloud service solution to establish a dedicated private network connection from your premises to AWS
	- essentially just connect your data center directly to AWS
- not traversing internet at all
- useful for high throughput workloads (i.e. a lot of network traffic) or need a stable and reliable secure connection
![dx](pic/dx.png)
![set_dx](pic/Set_dx.png)


### Global Accelerator
- improve availability and performance for local and global users
- includes following components
	- Static IP addresses
		- **provides two static IP addresses**
		- can bring your own IP address
	- Accelerator
		- directs traffic to optimal endpoints over AWS
		- includes one or more listener
	- DNS name
		- similar to `12312342abes.awsglobalaccelerator.com` - points to the static IP address that GA assigned to you
	- Network zone
		- services the static IP address for accelerator from a unique IP subnet
		- like AZ: isolated unit with its own physical infrastructure
		- by default, has two IP addresses. If one fails, can try another
	- listener
		- processes inbound connections from clients to GA
		- supports both TCP and UDP protocols
		- each listener has one or more endpoint groups associated with it
			- association by specifying the regions that you want to distribute traffic to
			- traffic is distributed to optimal endpoints within the endpoint groups associated with a listener
	- **endpoint group**
		- associated with a specific AWS region
		- includes one or more endpoints
		- can increase or reduce the percentage of traffic that would be otherwise directed to an endpoint group by adjust a setting called **traffic dial**
		- traffic dial lets you easily do performance testing or blue/green deployment testing for new releases
	- **endpoint**
		- can be network load balancer, application load balancer, EC2 instances, elastic IP address
		- can be internet-facing or internal
		- traffic is routed to endpoints based on configuration options (e.g. endpoint **weights**)

### Network costs
- use private IP addresses over public to save on costs
- if want to cut cost, group EC2 instances in the same AZ and use private IP addresses. This will be cost free, but beware of single point of failure

### Elastic Load Balancer
- 3 types of load balancer
	- **application load balancer**
		- best suited for load balancing HTTP and HTTPS traffic
		- operate at layer 7 and are application aware
		- can create advanced request routing, sending specified requests to specific web servers (by creating if-then rules)
	- **network load balancer**
		- best suited for load balancing of TCP traffic where extreme performance is required
		- operate at connection level 4
		- capable of handling millions of requests per second, while maintaining ultra-low latencies
	- **classic load balancer**
		- legacy elastic load balancer
		- low cost
		- can load balance HTTP(S) applications and use layer 7 specific features
		- can use strict layer 4 load balancing for applications that rely purely on the TCP protocol
- if application stops responding, the ELB responds with a **504 error**
	- means the application not responding within the idle timeout period
	- mean you need to trouble shoot the application - Web server or database server?
- if you need the IPv4 address of your end user, look for the **X-Forwarded-For** header
- no IP address for ELB; only DNS names (application and classic)
  - you can get a static IP address for network load balancer
- instances monitored by ELB are reported as `InService` or `OutofService`
- load balancer routes requests to the targets in a **target group**
	- you can set the targets and health checks
	- not needed for classic load balancer
- **sticky sessions** allow you to bind a user's session to a specific EC2 instance
	- ensures that all requests from the user during the session are sent to the same instance
	- e.g. online editing of a file: edits do not go to separate instances
	- can be useful if you are storing information locally to that instance
- **cross zone load balancing**
	- distribute traffic (evenly) across different AZ
![cross zone lb](pic/cross_zone_lb.png)
- **path patterns**: can create a listener with rules to direct traffic (forward requests) based on URL path
	- also known as path-based routing
	- e.g. route general requests to one target group and requests to render images to another target group

## Databases

### RDS: Relational Database System
- 6 RDS on AWS: SQL Server, Oracle, MySQL Server, PostgreSQL, Aurora, MariaDB
- 2 key features
	- **multi-AZ**
		- exact synchronous copy of production database in another AZ
		- for disaster recovery
		- fail-over is automatic
		- can force a fail-over by rebooting the RDS instance
		- not applicable to Aurora (due to its own unique fault-tolerant design)
	- **read replicas**
		- read-only copy of the production database; can have multiple (up to 5)
		- asynchronous replication from the primary RDS instance to the read replica
		- for performance (especially for read-intensive workload); not for disaster recovery
		- must have backups turned on to create read replicas
		- can have read replicas of read replicas
		- each read replica has its own DNS end point
		- can create read replicas of multi-AZ source databases
		- read replicas can be promoted to be their own databases. this breaks the replication
		- can have a read replica in a different region
		- not applicable to SQL Server
- RDS is for OLTP; Red Shift is for OLAP
	- OLTP: Online Transaction Processing: e.g. order number 323245
		- prefer provisioned IOPS (instead of standard storage)
	- OLAP: Online Analytics Processing: e.g. net profit for EMEA
- RDS runs on virtual machine; you don't have access to the operating systems (cannot SSH to it)
	- patching of RDS operating system and DB is Amazon's responsibility
- RDS is not serverless (except for Aurora Serverless)
- backups
	- **automated backup**
		- point-in-time recovery within a retention period
		- done in a scheduled window
		- same size as your database
		- enabled by default
	- **database snapshot**
		- initiated manually
		- stored after DB is terminated
	- restored version (either approach) will be a new RDS instance with a new DNS endpoint
- encryption
	- encryption at rest supported by all 6 RDS services
	- use AWS Key Management Service (KMS)
	- underlying data encrypted all together (including backups, read replicas) 
- RDS instance port number is automatically applied to RDS DB security group

### DynamoDB
- one of the NoSQL databases
- fully managed
- serverless
- stored on SSD storage
- spread across 3 geographically distinct data centers
- eventual consistent reads (default)
	- consistency across all copies reached within 1 second
- strongly consistent reads
	- returns a result that reflects all writes that received a successful response prior to the read
- DynamoDB Accelerator (DAX)
	- fully managed, highly available, in-memory cache
	- reduce request time
	- allow both read and write
- transaction: prepare/commit reads or writes for "all-or-nothing" operations (e.g. money transfer)
- on-demand capacity: 
	- pay-per-request pricing
	- no charge for read/write when idling - only storage and backups
	- higher cost per request than with provisioned capacity
- on-demand backup and restore
	- full backups at any time
	- zero impact on table performance or availability
	- consistent within seconds and retained until deleted
	- operates within same region as the source table
- point-in-time recovery (PITR)
	- protects against accidental writes or deletes
	- restore to any point in the last 35 days
	- incremental backups
	- not enabled by default
	- latest restorable: 5 minutes in the past
- streams
	- time-ordered sequence of item-level changes in a table
	- shard: a collection of stream records (data)
	- stored for 24 hours
- global tables
	- managed multi-master, multi-region replication
	- globally distributed applications
	- based on DynamoDB streams
	- multi-region redundancy for disaster recovery
	- no application rewrites
	- replication latency under 1 second
- security
	- encrytion at rest using KMS
	- site-to-site VPN
	- Direct Connect
	- IAM policies and roles
	- fine-grained access
	- CloudWatch and CloudTrail
	- VPC endpoints

### Redshift
- fully managed data warehouse service for BI
- Configuration
	- single node (160Gb)
	- multi-node
		- leader node (manages client connections and receives queries)
		- compute node (store data and perform computations) - up to 128 compute nodes
- advanced compression
	- compress data based on column (instead of row)
	- doesn't require indexes or materialized views
- massive parallel processing (MPP)
	- automatically distrbutes data and query load across nodes
- backups
	- enabled by default with a 1-day retention period
	- max retention period is 35 days 
	- Redshift always attempts to maintain 3 copies of your data (original and replica on compute node and a backup in S3)
	- asynchronously replicate snapshots to S3 in another region for disaster recovery
- pricing
	- based on compute node hours: billed for 1 unit per node per hour
	- no charge for lead node
	- charge for backup; charge for data transfer within VPC
- security
	- in transit using SSL
	- at rest using AES-256 encryption
	- by default Redshift takes care of key management
		- manage your own key through HSM
		- AWS Key Management Service
- availability
	- only available in 1 AZ (no multi-AZ)
	- can restore snapshots to new AZs

### Aurora
- MySQL and PostgreSQL-compatible relational database engine
- better performance than traditional RDS and cheaper
- start with 10GB, scales in 10GB increments to 64TB (storage auto-scaling)
- compute resource up to 32vCPUs and 244GB of memory
- 2 copies of data contained in each AZ, with minimum of 3 AZ (6 copies) of data
- handles the loss of 
	- up to 2 copies of data without affecting database write availability
	- up to 3 copies of data without affecting database read availability
- self-healing: data blocks and disks are continuously scanned for errors and repaired automatically
- replicas
	- Aurora Replicas (15): automated failover available
	- MySQL Read Replicas (5)
	- PostgreSQL (1)
- in-region replica only
- automated failover with no data loss (with Aurora replicas only)
- backups
	- automated backups enabled; does not impact performance
	- snapshots can be taken; no impact on performance
	- snapshots can be shared with other AWS accounts
- Aurora Serverless
	- on-demand, auto-scaling configuration for SQL-compatible editions of Aurora
	- automatically starts up, shuts down and scales based on needs
	- option for infrequent, intermittent, or unpredictable workloads

### ElastiCache
- a web service that makes it easy to deploy, operate, and scale an in-memory cache in the cloud
- a way to speed up database performance by caching most commonly used content
	- e.g. top 10 most popular products on a retailing site
- Memcached vs. Redis
	- Redis is multi-AZ
	- can do backups and restores of Redis
![ElastiCache](pic/elasticache_2.png)

### DMS: Database Migration Service
- a cloud service to migrate databases
- essentially a server in AWS that runs replication software
- can migrate data into AWS, between on-premise instances, or between combinations of cloud and on-premise setups
- source database remains operational
- can pre-create target tables manually, or use AWS Schema Conversion Tool (SCT)
- types of migration
	- homogeneous (e.g. Oracle to Oracle) - don't need SCT
	- heterogeneous (e.g. SQL Server to Aurora) - need SCT
![DMS source target](pic/DMS_source_target.png)

### EMR: Elastic Map Reduce
- cloud big data platform
- central component is **cluster**, a collection of EC2 instances
- each instance is a node; each node has a role, referred to as the node type
- EMR installs different software components on each node type
	- master node: every cluster must have a master node; log data is stored on master node by default
	- core node: run tasks and store data
	- task node (optional): run tasks and does not store data 
- security: periodically archive the log files on master node to S3 (in case master node is terminated)
	- must perform step when setting up the cluster; cannot do so after setup


## Service and Applications

### CloudFormation
- a way of scripting cloud environment (mostly JSON)
- Quick Start has a collection of CloudFormation templates (e.g. set up a SharePoint server)
- more detailed and powerful than Elastic Beanstalk, but takes time to learn and configure

### Elastic Beanstalk
- can quickly deploy and manage applications in AWS without managing the infrastructure
	- no knowledge of AWS needed; can just upload application code and let AWS set up everything
- automatically handles capacity provisioning, load balancing, scaling, health monitoring
- you can modify settings (e.g. add auto scaling) after provision
- belongs to Compute section

### SQS
- a web service to give you access to a message queue
- distributed queue system that enables web service applications to quickly and reliably queue messages
- **pull-based**, not push-based
- message is 256 KB in size (can be bigger but will be saved to S3)
- messages can be kept in the queue from 1 minute to 14 days; the default retention period is 4 days
- SQS guarantees that your messages will be processed at least once
- can **decouple** the components of an application so they run independently
	- any component of a distributed application can store messages in a fail-safe queue
- a queue is a temporary repository for messages that are awaiting processing
	- acts as a buffer between the component producing and saving data, and the component receiving the data for processing
	- resolves issues that arise if the producer is faster, or if the production is only intermittent
- 2 types of queues
	- **standard queue**
		- nearly-unlimited number of transactions per second
		- guarantee a message is delivered at least once
		- provide best-effort ordering to ensure messages are generally delivered in the same order they are sent
		- may have different order and/or receiving duplicated messages
	- **FIFO queue** (first-in-first-out)
		- order is strictly preserved
		- a message is delivered only once and remains available until a consumer processes it and deletes it
		- duplicates are not introduced into the queue
		- support message groups that allow multiple ordered message groups within a single queue
		- limited to 300 transactions per second (TPS)
- **visibility timeout** is the amount of time that the message is invisible in the SQS queue after a reader picks up that message
	- maximum is 12 hours
	- if the job is processed before visibility timeout, the message will be deleted from the queue
	- if the job is not processed within that time, the message will become visible again and could result in a duplicated message delivery - potential cause of duplication (in exam questions)
- Amazon SQS **long polling** is a way to retrieve messages from your Amazon SQS queues
	- short polling returns immediately (even if the queue is empty)
	- doesn't return a response until a message arrives in the message queue, or the long poll times out
	- long polling is an efficient and cost-saving option if queue tends to be empty
- message-oriented API
- need to implement your own application-level tracking, especially if you have multiple queues

### SWF: Simple Workflow Service
- a web service to coordinate work across distributed application components
- enables applications of a wide range of use cases to be designed as a coordination of tasks
- **tasks** represent invocations of various processing steps in an application
	- executable code, web service calls, human actions, scripts
- combine digital environment with **human actions**
	- e.g. place an order on Amazon -> pick up product from warehouse
- workflow executions can last up to 1 year
- task-oriented API
- ensures a task assigned only once and is never duplicated
- keeps track of all the tasks and events in an application
- SWF actors
	- Workflow Starters: an application to initiate a workflow
	- Deciders: control the flow of activity tasks
	- Activity Workers: carry out the activity tasks

### SNS: Simple Notification Service
- set up, operate and send notifications
- push cloud notifications
	- to all operating systems
	- via SMS
	- by email
	- to any HTTP endpoint
- instantaneous push-based delivery
- can group multiple recipients using topics
	- a topic is an "access point" for allowing recipients
- stored redundantly across multiple AZs
- simple APIs and easy integration with applications
- inexpensive, pay-as-you-go model with no upfront cost
- web-based AWS management console offers the simplicity of a point-and-click interface
- SNS vs. SQS:
	- both are messaging service
	- SNS: **push-based**; SQS: pull/poll-based

### Elastic Transcoder
- media transcoder in the cloud
- convert media files from original source format into different formats on target device (e.g. smartphones, PCs, etc.)
- provides transcoding presets for popular output formats; don't need to guess which setting works best
- pay based on the minutes transcoded and the resolution


### API Gateway
- fully managed service for developers to publish, maintain, monitor and secure APIs at any scale
- essentially a doorway to your applications
- can do the following
	- expose HTTPS endpoints to define a restful API
	- serverlessly connect to services like Lambda
	- send each API endpoint to a different target
	- connect to CloudWatch to log all requests for monitoring
	- maintain multiple versions of your API
- run efficiently with low cost
- scale effortlessly and automatically
- track and control usage by API key
- throttle requests to prevent attacks
- how to configure API gateway
	- define an API (container)
	- define resources and nested resources (URL paths)
	- for each resource:
		- select supported HTTP methods
		- set security
		- choose targets (e.g. EC2, Lambda, DynamoDB)
		- set request and response transformations
- deploy API to a stage
	- use API gateway domain by default
	- can use custom domain
	- now supports AWS Certificate Manager: free SSL/TLS certs
- can enable API **caching** to cache your endpoint's response
	- can reduce the number of calls made to your endpoint and improve latency of the requests
	- when enabled, API Gateway caches responses from your endpoint for a specific time-to-live (TTL) period in seconds
	- API Gateway then responds to the request by looking up the endpoint response from the cache instead of making request to endpoint
- same-origin policy
	- a web browser permits scripts contained a first web page to access data in a second web page, but only if both web pages have the same origin
	- this is done to prevent cross-site scripting (XSS) attacks
		- enforced by web browsers
		- ignored by tools like PostMan and curl
- **Cross-origin resource sharing (CORS)** is one way the server can relax the same-origin policy
	- allow restricted resources (e.g. fonts) on a web page to be requested from another domain outside the first resource's domain
	- if you see an error of "Origin policy cannot be read at the remote resource", it means you need to enable CORS on API Gateway
	- CORS is enforced by client (e.g. browser)

### Kinesis
- streaming data
	- data generated continuously by many data sources, typically sent in the data records simultaneously and in small sizes
	- e.g. purchases from online stores
- Kinesis is a platform to send your streaming data to
- 3 types Kinesis
	- **Kinesis Streams**
		- storage of 24 hours by default, up to 7 days
		- consists of **shards**
			- 5 transactions per second for reads; 
			- data read rate up to 2MB per second
			- up to 1000 records per second for writes
			- data write rate up to 1 MB per second (including partition keys)
		- data capacity of stream is a function of the number of shards -> sum of the shards' capacity
	- **Kinesis Firehose**
		- no data persistence: need to process data immediately
		- optional: Lambda function to analyze on the fly
	- **Kinesis Analytics**
		- works with both Streams and Firehose
		- analyze data on the fly inside either Kinesis service (instead of using a Lambda function)
		- then saves data in downstream service

### On-premise service
- Database migration service (DMS)
	- allows you to move databases to and from AWS
	- supports both homogeneous and heterogeneous migration
- Server Migration Service (SMS)
	- incremental replication of your on-premise servers
	- can be used as a backup tool, multi-site strategy, and a disaster recovery tool
- AWS Application Discovery Service
	- helps enterprise to plan migration projects by gathering info about its on-premise data center
	- how it works:
		1. install AWS Application Discovery Agentless Connector as a virtual appliance on VMware
		1. it then builds a server utilization map and dependency map of the on-premise environment
		1. the collected data is retained in encrypted format; can be exported as csv file to estimate total cost of ownership (TCO)
	- this data is also available in AWS Migration Hub, where you can migrate the discovered servers and track migration progress
- VM Import/Export
	- migrate existing applications into EC2
	- can be used to create a DR strategy on AWS or use AWS as a second site
	- can use it to export your AWS VMs to your on-premise data center
- download Amazon Linux 2 as an ISO


### SAM: Serverless Application Model
- SAM: CloudFormation extension optimized for deploying serverless applications
- can define functions, APIs, tables
- supports anything CloudFormation supports
- can run serverless applications locally (in Docker)
- can package and deploy using CodeDeploy
![SAM](pic/SAM.png)


### ECS: Elastic Container Service
- a **container** is a package that contains an application, libraries, runtime and tools required to run it
- a container runs on a container engine like **docker**
- kind of like virtual machine, but different
	- provides the isolation benefits of virtualization, but with less overhead and faster starts than VMs
	- containerized applications are portable and offer a consistent environment
![container](pic/Container.png)
- ECS is a managed container **orchestration** service
	- can create clusters to manage fleets of container deployments
	- can manage EC2 or Fargate instances
		- Fargate: serverless container engine; eliminates needs to manage servers; each workload runs in its own kernel; works with both ECS and EKS
		- choose EC2 for compliance requirements, broader customization, GPUs
	- schedules containers for optimal placement
	- define rules for CPU and memory requirement
	- monitor resource utilization
	- deploy, update, roll back containers
	- integrates with VPC, security groups, EBS volumes, ELB
	- supports CloudTrail and CloudWatch
	- free service
- ECS components
![ECS](pic/ECS_components.png)
- EKS: Elastic Kuernete Service
	- K8s is an open source software to deploy and manage containerized applications at scale
	- same toolset on-premises and in cloud
	- containers are grouped in pods
	- supports both EC2 and Fargate
	- use case:
		- already using K8s
		- want to migrate to AWS
- ECR: managed docker container registry
	- store, manage, and deploy images
	- integrated with ECS and EKS
	- works with on-premises deployments
	- highly available
	- integrated with IAM
	- pay for storage and data transfer
- ECS security: task role vs. ECS instance role
![ECS security](pic/ECS_security.png)


## Best Practice Design

### HPC: High Performance Computing
- data transfer
	- Snowball, Snowmobile
	- DataSync
	- Direct Connect
- cache
	- CloudFront
	- API Gateway
	- ElastiCache - Memcached and Redis
	- DynamoDB Accelerator (DAX)
- compute and network
	- EC2 instances (GPU or CPU optimized)
	- EC2 fleets (e.g. spot fleets)
	- placement groups (cluster placement groups for low latency)
	- enhanced networking (ENA, VF, EFA)
- storage
	- instance-attached storage
		- EBS
		- instance store
	- network storage
		- S3: distributed object-based storage; not a file system
		- EFS: scale IOPS based on total size, or use provisioned IOPS
		- FSx for Lustre: HPC-optimized distributed file system; millions of IOPS; backed by S3
- orchestration and automation
	- AWS Batch: run many batch computation jobs
	- AWS ParallelCluster

### HA: Highly Available architecture
- always design for failure
- use multiple AZs and regions wherever you can
- multi-AZ vs. read replicas for RDS
- scaling out vs. scaling up
	- scaling out: use auto scaling groups (add instances)
	- scaling up: increase resources inside EC2 instance (upgrade RAM or CPU)
- beware of cost element
- know different S3 storage classes
![HA](pic/HA_example.png)
- HA bastion hosts
	- option 1: separate hosts in separate AZ; use a network load balancer with static IP address and health checks
		- cannot use application load balancer, because it is layer 7 and we need layer 4
		![HA_bastion1](pic/HA_bastion_1.png)
	- option 2: one host in a AZ behind an auto scaling group with health checks and a fiexed EIP. 
		- if the host fails, the health check will fail and the auto scaling group will automatically provision a new EC2 instance
		- not 100% fault tolerant; will have some downtime
		- lowest cost option
		![HA_bastion2](pic/HA_bastion_2.png)


