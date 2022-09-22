# AWS-Solutions-Architect-Associate (C00-003)
___
# Computing

## Elastic Compute Cloud (EC2)
- Regional Service
- EC2 (Elastic Compote Cloud) is an Infrastructure as a Service
- Stop & Start of an instance may change its public IP but not its private IP
- <b>AWS Compute Optimizer</b> recommends optimal AWS compute resource for your workloads
- There is a vCPU-based On-demand instance soft limit per region

### EC2 User Data
- It is possible to bootstrap our instances using an EC2 User data script
- bootstrapping means launching commands when a machine starts
- Scripts is only run once at the instance first start
- EC2 user data is used to automate boot task such as: 
  - Installing updates
  - Installing software
  - Downloading commons files from the internet
- EC2 User data scripts runs with root user privilege

### Instance Types
- General purpose
  - Great for diversity of workloads such as web servers or code repositories
  - Balance between compute, memory and networking
- Compute Optimized
  - Great for compute intensive task
    - Batch processing
    - Media transcoding
    - High performance computing (HPC)
    - web or gaming servers
- Memory Optimized
  - Fast performance for workloads that process large data sets in memory
  - Great for in-memory databases or distributed web caches
- Storage Optimized
  - Great for storage intensive task (accessing local databases)
  - OLTP systems (uses traditional DBMS)
  - Distributed File System (DFS)

### Security Group
- Security groups only contains allow rules
- Security groups rules can reference a resource by IP or Security Group
- External firewall for EC2 instances (if a request is blocked by SG, instance will never know)
- Security groups are stateful, this means any connection initiated successful will be completed.
- Default SG
  - all inbound traffic is blocked
  - inbound traffic from the same SG is allowed
  - all outbound traffic is allowed
- SG can be attached to multiple instances and vice versa
- Limited to a VPC (and hence to a region)
- Recommended to maintain a separate security group for SSH access
- Blocked request will give a <b>Time out</b> error

### Purchasing Options
- On-demand Instances
  - Pay for what you use (no upfront payment)
  - Has the highest cost
  - No long term commitment
  - Recommended for a <b>short-term and un-interrupted workloads</b>
- Standard Reserved Instances
  - Reservation Period: 1 year(+discount) or 3 years(+++discount)
  - Payment options - No Upfront(+), Partial Upfront(++), All Upfront(+++)
  - Recommended for steady state applications (think database)
  - Sell unused instances on the Reserved Instance Marketplace
  - Reserved Instance's Scope - Regional or Zonal (reserve capacity in an AZ)
- Convertible Reserved Instance
  - Can change the instance type
  - lower discount than Standard Reserved Instance
  - Cannot sell unused instances on the Reserved Instance Marketplace
- Schedule Reserved Instances
  - Reserved for a time window (ex. everyday from 9AM to 5PM)
- Spot Instances
  - Work on a bidding basis where you are willing to pay a specific max hourly rate for the instances. Your instance can terminate if the spot price increase.
  - <b>Spot block</b> are designed not to be interrupted
  - Good for workloads that are resilient to failure
    - batch / distributed jobs
    - data analysis
    - image processing
- Dedicated Hosts
  - Server hardware is allocated to a specific company
  - 1 or 3 year reservation period
  - Useful for software that have complicated licensing model (BYOL - bring you own license) or for companies that have a strong regulatory or compliance needs. 
  - Most expensive option
- Dedicated Instances
  - Dedicated hardware
  - May share hardware with other instances in same account
  - No control over instance placement
- Capacity Reservation
  - Ensure you have the available capacity in a specific AZ for any duration
  - No time commitment, no billing discount
  - You're charged at On-demand rate whether you run instances or not
  
### Spot Instances
- Spot Requests
  - <b>One-time</b>: Request once opened, spins up the spot instances and the request closes.
  - <b>Persistent</b>:
    - Request will stay disabled while the spot instances are up and running
    - It becomes active after the spot instance is interrupted.
    - If you stop the spot instance, the request will become active only after you start the spot instance.
  - You can only cancel spot instance request that are open, active, or disabled.
  - Cancelling a Spot Request doest not terminate instances. you must first cancel a Spot Request, and then terminate the associated Spot Instances.
- Spot Fleets
  - Combination of spot and on-demand instances(optional) that tries to optimize for cost or capacity
  - Launch Templates must be used to have on demand instances in the fleet
  - Can consist of instances of different classes
  - Strategies to allocate Spot Instances:
    - lowestPrices : from the pool with lowest price (cost, optimization, short workload)
    - diversified: distributed across all pools(great for availability, long workloads)
    - capacityOptimized: pool with the optimal capacity for the number of instances.
### Elastic IP
- Static public IP that you own as long as you don't delete it
- Can be attached to an EC2 Instance(even when it is stopped)
- Soft limit of 5 elastic IPs per account
- Doesn't incur charges as long as the following conditions are met
  - is associated with an Amazon EC2 instance
  - instance associated with the elastic IP is running
  - the instance has only one Elastic IP attached to it

### Placement Groups
- Cluster Placement Group (optimize for network)
  - All the instances are placed on the same hardware (same rack)
  - Pros: Great network(10 Gbps bandwidth between instances)
  - Cons: If the rack fails, all instances will fail at the same time
  - Used in HPC (minimize inter-node latency & maximize throughput)
- Spread Placement Group (maximize availability)
  - Each instance is in a separate rack (physical hardware) inside an AZ
  - Supports Multi AZ
  - Up to 7 instances per AZ per placement group (ex. for 15 instances, need 3 AZ)
  - Used for critical applications with need to high availability
- Partition Placement Group (balance of performance and availability)
  - Instances in a partition share rack with each other
  - If the rack goes down, the entire partition goes down
  - Up to 7 partitions per AZ
  - Used in <b>Big Data</b> applications (Hadoop, HDFS, HBase, Kafka, Cassandra)
 
> If you received a capacity error when launching an instance in a placement group that already has running instances, stop and start all of the instances in the placement group, and try the launch again. 
> Restarting the instances may migrate them to hardware that has a capacity for all the requested instances.
  
### Elastic Network Interfaces (ENI)
- ENI is a virtual network card that <b>gives a private IP to an EC2 instance </b>
- A primary ENI is created and attached to the instance upon creation and will be deleted automatically upon instance termination.
- We can create additional ENIs and attach them to an EC2 instance to access it via multiple private IPs.
- We can detach & attach ENIs across instances
- ENIs are tied to the subnet hence to the AZ

### EC2 Instance States
- Stop
  - EBS root volume is preserved
- Terminate
  - EBS root volume is destroyed
- Hibernate
  - Hibernate save the contents from the instance memory(RAM) to the EBS root volume
  - EBS root volume is preserved
  - The instance boots much faster as the OS is not stopped and restarted
  - When you start your instance:
    - EBS root volume is restored to its previous state
    - RAM contents are reloaded
    - Processes that were previously running on the instance are resumed
    - Previously attached data volumes are reattached and the instance retain its instance ID
  - Should be used for application that take a long time to start
  - <b>Not supported for Spot Instance </b>
  - Max hibernation duration = <b>60 days</b>
- Standby
  - Instance remains attached to the ASG but is temporarily put out of service(the ASG doesn't replace this instance)
  - Used to install updates or troubleshoot a running instance.

### EC2 Nitro
- Newer virtualization technology for EC2 Instances
- Better networking options (enhanced networking, HPC, IPv6)
- Higher speed EBS (64,000 EBS input/output operation per seconds (IOPS) max on Nitro instances whereas 32,000 on non-nitro)
- Better underlying security

### vCPU & Threads
- vCPU is the total number of concurrent threads that can be run on an EC2 instance
- Usually 2 threads per CPU core (eg 4CPU cores -> 8vCPU)

### Amazon Machine Image (AMI)
- AMIs are the image of the instance after installing all the necessary OS, software and configure everything
- it boots much faster because the whole thing is pre-packaged and doesnt have to be installed separately for each instance.
- Good for static configurations
- <b>Bound to a region</b> and can be copied across regions
- You can launch EC2 instances from:
  - A public AMI: AWS provided
  - Your own AMI: you make and maintain them yourself
  - An AWS Marketplace AMI: an AMI someone eles made(and potentially sells)

> When the new AMI is copied from region A into region B, it automatically creates a snapshot in region B because AMIs are based on the underlying snapshots.

### Instance Metadata
- URL to fetch metadata about the instance (http://169.254.169.254/latest/meta-data)
- This URL is internal to AWS and can only be hit from the instance

### EC2 Classic & ClassicLink
- Instances run in single network shared with other customers(this is how AWS started)
- <b>ClassicLink</b> allow you to link EC2-Classic instances to a VPC in your account

### Billing
- Reserved instances will be billed regardless of their state (billed for a reservation period)
- On-demand instances in `stopping` state when preparing to hibernate will be billed
- If an instance is running, it will be billed
- In all the other cases, an instance will not be billed

### Run Command
- System Manager <b>Run command</b> lets you remotely and securely manage the configuration of your managed instances. A managed instance is any EC2 instance that has been configured for <b> Systems Manager </b>
- Run command enables you to <b>automate common administrative tasks</b> and perform ad-hoc configuration changes at scale.
- You can use Run Command from the <b>AWS Console,</b> the AWS CLI, Tools for windows, powershell or AWS SDKs. Run command is offered at no additional cost.

### Instance Tenancy
- Default: Instance runs on shared hardware
- Dedicated: Instance runs on single-tenant hardware
- Host: Instance runs on dedicated host  
> - Tenancy of an instance can only be change from host to dedicated or dedicated to host after the instance has been launched
> - Dedicated instance tenancy takes precedence over default instance tenancy

### Troubleshooting
- The following are a few reason why instance might immediately terminate:
  - Reached the limit of EBS volume
  - EBS snapshot is corrupt
  - Root EBS volume is encrypted and you do not have permissions to access on the KMS key for decryption.
  - Instance store-backed AMI that you used to launch the instance is missing a required part

## Elastic Load Balancer (ELB)
- Regional Service
- Support Multi AZ
- Spread load across multiple EC2 Instances
- Separate public tracfic from private traffic
- Health check allow ELB to know which instances are working properly (done on port and a route, `/health` is common)
- <b>Does not support weighted routing</b>
> if no targets are associated with the target groups -> 503 Service Unavailable
> Using ALB & NLB, instances in peered VPCs can be used as targets using IP Addresses.

### Types
<b>Classic Load Balancer (CLB) - deprecated</b>
- Load balancing to a single application
- Supports HTTP, HTTPS (layer 7) & TCP, SSL (layer 3)
- Provides a fixed hostnames (`xxx.region.elb.amazonaws.com`)

<b>Application Load Balancer (ALB) </b>
- Load balancing to multiple applications(target groups)
- Operates at Layer 7 (HTTP, HTTPS and WebSocket)
- Provides a fixed hostname (`xxx.region.elb.amazonaws.com`)
- ALB terminates the upstream connection and creates a new downstream connection to the targets(connection termination)
- <b>Security Groups can be attached to ALBs</b> to filter requests
- Great for micro services & container-based applications (Docker & ECS)
- Client info is passed in the request headers
  - Client IP -> `X-Forwarded-For`
  - Client Port -> `X-Forwarded-Port`
  - Protocol -> `X-Forwarded-Proto`
- Target Groups
  - Health checks are done at the target group level
  - Target Groups could be
    - EC2 instances - HTTP
    - ECS task - HTTP
    - Lambda functions - HTTP request is translated into a JSON event
    - Private IP Addresses
- Listener Rules can be configured to route traffic to a different target groups based on: 
  - Path (example.com/users & exmaple.com/posts)
  - Hostname (one.example.com & other.example.com)
  - Query String (example.com/users?id=123&order=false)
  - Request Headers
  - Source IP address
  

<b>Network Load Balancer (NLB) </b>
- Operates at Layer 4 (TCP, TLS, UDP)
- Can handle millions of request per seconds (extreme performance)
- Lower latency ~ 100ms (vs 400 ms for ALB)
- 1 static public IP per AZ (vs a static hostname for CLB & ALB)
- Elastic IP can be assigned to NLB (helpful for whitelisting specific IP)
- Maintains the same connection from client all the way to the target
- <b>No security groups can be attached to NLBs. </b> They just forward the incoming traffic to the right target group as if those request were directly coming from client. so the <b>attached instances must allow TCP traffic on port 80 from anywhere</b>
- Within a target group, NLB can send traffic to
  - <b>EC2 Instances</b>
    - if you specify targets using an instance ID, traffic is routed to instance using the primary private IP address
  - <b>IP Addresses</b>
    - Used when you want to balance load for a physical server having a static IP.
  - <b>Application Load Balancer (ALB)</b>
    - Used when you want a static IP provided by an NLB but also want to use the features provided by ALB at the application layer.

<b>Gateway Load Balancer (GWLB)</b>
- Operates at layer 4 (Network Layer) - IP protocol
- Used to route requests to a fleet of 3rd party virtual appliances like Firewalls, Intrusion Detection and Prevention System (IDPS), etc
- Performs two functions:
  - Transparent Network Gateway (single entry/exit for all traffic)
  - Load Balancer (distributes traffic to a virtual appliances)
- Uses GENEVE protocol
- Target groups for GWLB could be
  - EC2 instances
  - IP address

### Sticky Sessions (Session Affinity)
- Request coming from client is always redirected to the same instance based on a cookie. After the cookie expires, the request coming from the same user might be redirected to another instance.
- <b>Only supported by CLB & ALB</b>
- Used to ensure the user doesn't lose his session data, like login or cart info, while navigating between web pages.
- <b>Stickiness may cause load imbalance</b>
- Cookies could be
  - Application-based (TTL defined by the application)
  - Load Balancer generated (TTL defined by the load balancer)

### Cross-zone Loading Balancing
- Allows ELBs in different AZ containing unbalanced number of instances to distributes the traffic evenly across all instances in all the AZ registered under a load balancer.
- Supported Load Balancers
  - Classic Load Balancer
    - Disabled by default
    - No charges for inter AZ data
  - Application Load Balancer
    - Always on (can't disabled
    - No charges for inter AZ data
  - Network Load Balancer
    - Disabled by default
    - Charges for inter AZ data

### In-flight Encryption
- Use an NLB with a TCP listener & terminate SSL on EC2 Instances
- Use an ALB with an HTTPs listener, install SSL certificates on the ALB & terminate SSL on the ALB
- Communication between ALB & EC2 instances can happen over HTTP inside the VPC
- <b>Server Name Indication (SNI)</b>
  - SNI allows us to load multiple SSL certificates on one Load Balancer to serve multiple websites securely
  - <b>Only works for ALB & CLB </b> (CLB only supports one SSL certificates)
  - <b>Supported in Cloudfront</b> also

### Connection Draining (De-registration delay)
- When an instance is to be de-registered from the ELB, the in-flight requests being served by that instance are given some pre-defined time to complete before the ELB de-register it.
- ELB stops sending new requests to the EC2 instance which is de-registering
- Set manually (0 to 3600 seconds) <b>(default : 300 secondss)</b>
> For instances behind an ELB and using ASG, increase the de-registration delay to ensure that the in-flight are completed before the ELB deregisters an instance which is to be terminated by the ASG.

### Access Logs 
- Captures detailed information about requests sent to the load balancer
- Used to analyze traffic patterns and troubleshoot issues
- Disabled by default

### Misc
- Security Group for a public facing ELB
  - ELB will be publicly available on the internet, so its security group should allow HTTP and HTTPS traffic from anywhere. EC2 should only allow traffic from the ELB, so its security group should allow HTTP requests from ELB's security group

## Auto Scaling Group (ASG)
- Regional Service
- Supports Multi AZ
- Automatically add or remove instances (scale horizontally) based on the load
- Free (pay for the underlying resources)
- IAM roles attached to an ASG will get assigned by an ELB (hence replace them)

> - Even if an ASG is deployed across 3 AZs, minimum number of instances to remain highly available is still 2
>
> - If you have an ASG with running instances and, you delete the ASG , The instances will be terminated and the ASG will be deleted.
> 

### Scaling Policies
- <b>Schedule Policies</b>
  - Scale based on a schedule (ex. create the min capacity to 10 at 5PM on Fridays)
  - Used when the load pattern is predictable
- <b>Simple Scaling</b>
  - Scale to certain size on a `CloudWatch` alarm
  - Ex. when CPU > 90%, then scale to 10 instances
- <b>Step Scaling</b>
  - Scale incrementally in steps using `CloudWatch` alarms
  - Ex. when CPU > 70%, then add 2 units and when CPU < 30%, then remove 1 unit
  - Specify the <b>instance warmup time</b> to scale faster
- <b>Target Tracking Scaling</b>
  - ASG maintains a `CloudWatch` metric and scale accordingly (automatically crate CW alarms)
  - Ex. maintain CPU usage at 40%
- <b>Predictive Scaling</b>
  - Historical data is used to predict the load pattern using ML and scale automatically

### Launch Configuration & Launch Template
- Defines the following info for ASG
  - AMI (Instance Type)
  - EC2 User Data
  - EBS Volumes
  - Security Groups
  - SSH Key Pair
  - Min / Max / Desired Capacity
  - Subnets (where the instances will be created)
  - Load Balancer (specify which ELB to attach instances)
  - Scaling Policy
- <b>Launch Configuration (legacy)</b>
  - Cannot be updated (must be re-created)
  - <b>Does not support Spot Instances</b>
- <b>Launch Template (newer)</b>
  - Versioned
  - Can be updated
  - <b>Supports both On-demand and Spot Instances</b>
  - Recommended by AWS

### Scaling Cooldown
- After a scaling activity happens, the ASG goes into cooldown period(default 300 seconds) 
during which it does not launch or terminate additional instances (ignores scaling requests) to allow the metrics to stabilize
- Use a ready to use AMI to launch instances faster to be able to reduce the cooldown period

### Health Checks
- By default, ASG uses the EC2 status check (not the ELB health check). this could explain why some instances that are labelled as unhealthy 
an ELB are still not terminated by the ASG.
- To prevent ASG from replacing unhealthy instances, suspend the <b>ReplaceUnhealthy</b> process type

> ASG creates new scaling activity for terminating the unhealthy instance and then terminates it. 
> Later, another scaling activity launches a new instance to replace the terminated instance.

### Termination Policy 
- Select the AZ with the most number of instances
- Delete the instance with the oldest launch configuration
- Delete the instance which is closes to the next billing hour
> - ASG does not immediately terminate instances with an <b>Impaired</b> status, it waits a few minutes for the instance to recover
> - ASG doesn't terminate an instance that came into service based on EC2 status checks and ELB health checks until the <b>health check grace period</b> expires.
 
### Re-balancing AZs
- ASG ensures that the group never goes below the minimum scale. Actions such as changing the AZ for the group or explicitly terminating
or detaching instances can lead to the ASG becoming unbalanced between AZs. In such cases, ASG compensates by <b>re-balancing</b> 
the AZs by <b>launching new instances before terminating the old ones,</b> so that re-balancing does not compromise the performance or availability of the application.

### Lifecycle Hooks
- Used to perform extra steps before creating or terminating an instance. 
  - Example: 
    - Install some extra software or do some checks (during pending state) before declaring the instance as `in service`
    - Before the instance is terminated (terminating state). extract the log files
  - <b>Without lifecycle hooks, pending and terminating states are avoided</b>

### Attach running instances to an existing ASG
- The running instance must meet the following criteria
  - The AMI used to launch the instance still exists
  - The instance is not a member of another ASG
  - The instance is launch into one of the AZ defined in your ASG

## Lambda

- Function as a Service (FaaS)
- Serverless
- Auto-scaling
- Pay per request (number of invocations) and compute time
- Integrated with `CloudWatch` for monitoring
- <b>Not good for running containerized application </b> 
- Can package and deploy Lambda functions as container images

### Performance
- Increase RAM will improve CPU and network
- RAM: 128 MB - 10GB
- <b>Max execution time: 15 mins</b>
- Disk capacity in function container (`/tmp`): 512 MB
- Environment variables: 4KB

### Deployment
- Deployment size
  - Compressed: 50 MB
  - Uncompressed: 250 MB
- If you intend to reuse code in more than one Lambda function, consider creating a
<b>Lambda Layer</b> (a ZIP archive that contains libraries) for the reusable code. With Layers,
You can <b>use libraries in your function without including them in the deployment package.</b>
Layers let you keep you deployment package small, which makes deployment easier. A function can use up to 
5 layers at a time.

### Networking
- By default, <b>Lambda function operates from an AWS-owned VPC</b> and hence have <b>access to any public internet</b>
or <b>public AWS API</b> (ex. lambda functions can interact with AWS `DynamoDB` APIs to `PutItem`)
- Once a Lambda function is <b>VPC-enabled</b>, all network from your function is subject to 
the routing rules of your VPC/Subnet. If your function needs to interact with a public resource, it will need a
<b>`NAT Gateway`</b>. You should only enable your functions for VPC access when you need to interact with
a private resource located in a private subnet (ex. RDS databse)

> To enable your Lambda function to access resources inside your private VPC, you must provide <b>VPC subnet IDs</b> and 
<b>Security Group IDs</b>. AWS Lamda uses this information to set up ENIs.

### Supported Languages
- Node.js
- Python 
- Java
- C#
- Golang
- Ruby
- Any other language using Custom Runtime API (community supported,Rust)

### Use cases
- Serverless thumbnail creation using S3 & Lambda
- Serverless CRON job using EventBridge & Lambda

### Lambda@Edge
- Deploy Lambda function alongside your `CloudFront` CDN for computing at edge locations
- Customize the CDN content using Lambda at the edge location (responsive)
- No server management (Lambda is deployed globally)
- Pay for what you see (no provisioning)
- Can be used to modify `CloudFront` request & responses.
- We can create a global application using `Lambda@Edge` where `S3` hosts a static website
which uses client side JS to send requests to `CloudFront` which will process the request in a lambda function
in that edge location to perform some operation like fetching data from `DynamoDB`.

## Step Functions
- Used to build <b>serverless workflows</b> to <b>orchestrate Lambda Functions</b>
- Represent flow as a <b>JSON state machine</b>
- Maximum workflow execution time: 1 year
- Features: sequence, parallel, conditions, timeouts, error handling. etc
- Provides a <b>visual graph</b> showing the current state and which path workflow has taken

### Simple Wroklow Service (SWF)
- Outdated service (step functions are preferred instead)
- Code runs on EC2 (not serverless)
- <b>Ensures that a task is never duplicated</b> (could replace standard SQS queues)
- 1 year max runtime
- <b>Built-in human intervention step</b>
- Step functions are recommended to be used for new applications, except: 
  - If you need external signals to intervene in the processes
  - If you need child processes that return values to parent processes
___

# Storage
## Instance Store
- <b>Hardware storage directly attached to EC2 instance</b> (cannot be detached and attached to another instance)
- <b>Highest IOPS</b> of any available storage (millions of IOPS)
- Ephemeral storage (losses data when instance is stopped, <b>hibernated</b> or terminated)
- Good for buffer / cache / scratch data / temporary content
- AMI created from an instance does not have its instance store volume preserved

> You can specify the instance store volumes only when you launch an instance. You can't attach instance store volumes
> to an instance after you've launched it
 
### RAID
- <b>RAID 0</b>
  - Improved performance of a storage volume by distributing reads & writes in a stripe across attached volumes
  - If you add a storage volume, you get the straight addition of throughput and IOPS
  - For high performance applications
- <b>RAID 1</b>
  - Improved data availability by mirroring data in multiple volumes
  - For critical applications

## Elastic Block Storage (EBS)
- Volume <b>Network Drive</b> (provides low latency access to data)
- Can only be mounted to 1 instance at a time (except EBS multi-attach)
- <b>Bound to an AZ</b>
- Must provision capacity in advance (size in GB & throughput in IOPS)
- By default, upon instance termination, the root EBS volume is deleted and any other attached EBS volume is not deleted
(can be over-ridden using `DeleteOnTermination` attributes)
- To replicate an EBS volume across AZ or region, need to copy its snapshot
- EBS multi-attach allows the same EBS volume to attach to multiple EC2 instances <b>in the same AZ</b>

> `DeleteOnTermination` attributes can be updated for the root EBS volume only from the CLI

### Volume Type

#### General Purpose SSD
- Good for system boot volumes, virtual desktops
- Storage: 1GB - 16 TB
- **gp3** 
  - **3,000 IOPS baseline** (max 16,000 - independent of size)
  - 125 MiB/s throughput (max 1000 MiB/s - independent of size)
- **gp2**
  - **Burst IOPS up to 3,000**
  - **3 IOPS per GB**
  - **Max IOPS: 16,000** (at 5,334 GB)

#### Provision IOPS SSD
- Optimized for **Transaction-intensive Applications** with high frequency of **small & random IO operations.**
They are sensitive to increased I/O latency.
- Maintain high IOPS keeping I/O latency down by maintaining a **low queue length** and a high number of IOPS available to the volume.
- **Supports EBS Multi-attach** (not supported by other types)
- **io1** or **io2**
  - Storage: 4GB - 16 TB
  - Max IOPS: **64,000 for Nitro EC2 Instances & 32,000 for non-Nitro**
  - **50 IOPS per GB** (64,000 IOPS at 1,280 GB)
  - io2 have more durability and more IOPS per GB (at the assme price as io1)
- **io2 Block Express**
  - Storage: 4GiB - 64 TB
  - Sub-millisecond latency
  - Max IOPS: 256,000
  - 1000 IOPS per GB

#### Hard Disk Drives (HDD)
- Optimized for <b>Through-put intensive Applications</b> that require <b>large & sequential IO operation</b>
and are less sensitive to increase I/O latency (big data, data warehousing, log processing)
- Maintain high throughput to HDD-backed volumes by maintaining a <b>high queue length</b> when 
performing a large, sequential I/O
- <b>Cannot be used as boot volume</b> for an EC2 instance
- Storage: 125 MB - 16 TB
- <b>Throughput Optimized HDD (st1)</b>
  - Optimized for large sequential read and writes (Big data, Data Warehouses, Log processing)
  - <b>Max throughput: 500 MB/s</b>
  - <b>Max IOPS: 500</b>
- <b>Cold HDD (sc1)</b>
  - For infrequently accessed data
  - Cheapest
  - <b>Max throughput: 250: MB/s</b>
  - <b>Max IOPS: 250</b>

### Encryption
- Optional
- For Encrypted EBS volumes
  - Data at rest is encrypted
  - <b>Data in-flight between the instance and the volume is encrypted</b>
  - All the snapshots are encrypted
  - All volumes created from the snapshot are encrypted
- Encrypt an un-encrypted EBS volume
  - Create an EBS snapshot of the volume
  - Copy the EBS snapshot and encrypt the new copy
  - Create a new EBS volume from the encrypted snapshot (the volume will be automatically encrypted)

> All EBS types and all instance families support encryption but not all instance types support encryption.

### Snapshot
- <b>Data Lifecycle Manager (DLM)</b> can be used to automate the creation, retention and deletion of snapshots of EBS volumes.
- Snapshots are incremental
- Only the most recent snapshot is required to restore the volume.

## Elastic File System (EFS)
- AWS managed Network File System (NFS)
- Can be mounted to multiple EC2 instances <b>across AZs</b>
- <b>Pay per use</b> (no capacity provisioning)
- <b>Autoscaling</b> (up to PBs)
- Compatible with <b>Linux</b> based AMIs (<b>POSIX</b> file system)
- <b>Uses security group to control access to EFS</b>
- Lifecycle management feature to move files to <b>EFS-IA</b> after N days

### Performance Mode
- General Purpose(default)
  - latency-sensitive uses cases (web server, CMS, etc.)
- Max I/O
  - higher latency & throughput (big data, media processing)

### Throughput Mode
- Bursting (default)
  - Throughput: 50MB/s per TB
  - Burst of up to 100MB/s
- Provisioned
  - Fixed throughput (provisioned)
### Storage Tiers
- Standard - for frequently accessed files
- Infrequent access (EFS-IA) - cost to retrieve files, lower price to store

### Security
- `EFS Security Group` to control network traffic
- `POSIX Permissions` to control access from hosts by IAM User or Group

## Simple Storage Service (S3)
- Global Service
- Object store (key-value pairs)
- <b>Buckets must have a globally unique name</b>
- Objects have a key (full path to the object): `s3://my_bucket/my_folder/another_folder/my_file.txt`
- The key is composed of bucket + `prefix` + <b>object name</b> + s3://my_bucket/my_folder/another_folder/my_file.txt
- There's no concept of directories within buckets (just keys with very long names that contains slashes)
- <b>Max object size: 5TB</b>
- Durability: 99.99999999999% (total 11 9's)
- <b>SYNC</b> command can be used to <b>copy data between buckets</b>, possibly in <b>different regions</b>
> - S3 delivers <b>strong read after-write consistency</b> (if an object is overwritten and immediately read, `S3` always 
> returns the latest version of the object)
> - S3 is strongly consistent for all `GET`, `PUT` and `LIST` operation

### Bucket Versioning
- Enabled at the bucket level
- Protects against unintended deletes
- Ability to restore to a previous version
- Any file that is not versioned prior to enabling versioning will have version `null`
- Suspending version does not delete the previous versions, just disables it for the future
- To restore a deleted object, delete it's `delete marker`
> - Versioning can only be suspended once it has enabled.
> - Once you version-enable a bucket, it can never return to an un-versioned state

### Encryption
- Can be enabled at the bucket level or a the object level
- #### Server Side Encryption (SSE)
  - SSE-S3
    - Keys managed by S3
    - AES-256 encryption
    - HTTP or HTTPS can be used
    - Must set header: `"x-amz-server-side-encryption":"AES256"`
  - SSE-KMS
    - Keys managed by KMS
    - HTTP or HTTPS can be used
    - KMS provides control over who has access to what keys as well as audit trails
    - Must set header: `"x-amz-server-side-encryption":"aws:ksm"`
  - SSE-C
    - Keys managed by the client
    - Client sends the keys in HTTPS headers for encryption/decryption (S3 discards the key after the operation)
    - <b>HTTPS must be used</b> as key (secret) is being transferred
- #### Client Side Encryption
  - Keys managed by the client
  - Client encrypts the object before sending it to S3 and decrypts it after retrieving it from S3

> <b>Default Encryption</b>: encrypt the files using the default encryption (specify the encryption for the file while uploading to override the default)
> Bucket policy can be used to force a specific type of encryption on the objects uploaded to `S3`

### Access Management 
- #### User based security
  - IAM policies defined which API calls should be allowed for a specific user
  - Preferred over bucket policy for <b>fine-grained access control</b>
- #### Resource based security (Bucket Policy)
  - Grant public access to the bucket
  - Force objects to be encrypted at upload
  - Cross-account access
  - Object access Control List (ACL) - applies to the objects while uploading
  - Bucket access Control List (ACL) - access policy that applies to the bucket

> By default, an S3 object is owned by the account that uploaded it even if the bucket is owned by another
> account. To get full access to the object, the object owner must explicitly grant the bucket owner access.
> As a bucket owner, you can create a bucket policy to require external users to grant `bucket-owner-full-control` when 
> uploading objects so the bucket owner can have full access to the objects.

### S3 Static Websites
- Host static websites (may contain client-side scripts) and have them accessible on the public internet over
<b>HTTP only</b> (for HTTPS, use `CloudFront` with S3 bucket as the origin)
- The website URL will be either of the following:
  - `<bucket-name>.s3-website-<region>.amazonaws.com`
  
- If you get a `403 (forbidden)` error, make sure the bucket policy allows public reads
- For cross-origin access to the `S3` bucket, we need to enable `CORS` on the bucket
- To host an S3 static website on a custom domain using `Route 53`, the bucket name should be the same as your domain
or subdomain `portal.tutorialsdojo.com`, the name of the bucket must be `portal.tutorialsdojo.com`

### MFA Delete
- MFA required to
  - permanently delete on object version
  - suspend versioning on the bucket
- <b>Bucket Versioning must be enabled</b>
- can only be enabled or disabled by root user

### Server Access Logging
- Most detailed way of logging access to S3 buckets (better than `CloudTrail`)
- Does not support <b>Data Events</b> & <b>Log File Validation</b> (use `CloudTrail` for that)
- Store S3 access logs into another bucket
- Logging bucket should not be the same as monitored bucket (logging loop)

### Replication
- <b>Asynchronous replication</b>
- Objects are replicated with the <b>same version ID</b>
- Supports <b>cross-origin</b> and <b>cross-account</b> replication
- <b>Versioning must be enabled for source and destination buckets</b>
- For DELETE operations:
  - Replicate delete markers from source to target (optional)
  - Permanent deletes are not replicated
- There is <b>no chaining of replication</b>. So, if bucket 1 has replication into bucket 2, which has replication into bucket 3,
Then objects created in bucket 1 are not replicated to bucket 3.

### Pre-signed URL
- Pre-signed URLs for S3 have temporary access token as query string parameters which allow anyone with the URL
to temporarily access the resource before the URL expires (default 1 hour)
- Pre-signed URLs inherit the permission of the user who generated it
- Uses:
  - Allow only logged-in users to download a premium video
  - Allow users to upload files to a precise location in the bucket
  
### Storage Classes
- Data can be transitioned between storage classes manually or automatically using lifecycle rules
- Data can be put directly into any storage class
- <b>Standard</b>
  - 99.99% availability
  - Most Expensive
  - Instant retrieval
  - No cost on retrieval (only storage cost)
  - For frequently accessed data
- <b>Infrequent Access</b>
  - For data that is infrequently accessed, but requires rapid access when needed
  - Lower storage cost than Standard but <b>cost on retrieval</b>
  - Can <b>move data into IA from Standard only after 30 days</b>
  - Two types:
    - <b>Standard IA</b>
      - 99.9% availability
    - <b>One-Zone IA</b>
      - 99.5% availability
      - Data is lost if AZ fails
      - Storage for infrequently accessed data that can be easily recreated
- <b>Glacier</b>
  - For data archival
  - Cost for storage and retrieval
  - <b>Can move data into Glacier from Standard anytime</b>
  - Objects cannot be directly accessed, they first need to be restored which could take some time (depending on the tier) to fetch the object
  - <b>Default encryption for data at rest and in-transit</b>
  - Three Types:
    - <b>Glacier Instant Retrieval</b>
      - 99.9% availability
      - Millisecond retrieval
      - Minimum storage duration of <b>90 days</b>
      - Great for data accessed once a quarter
    - <b>Glacier Flexible Retrieval</b>
      - 99.99% availability
      - 3 retrieval flexibility (decreasing order of cost):
        - Expedited (1 to 5 minutes)
          - Can <b>provision retrieval capacity</b> for reliability
          - Without provisioned capacity expedited retrievals might be rejected in situations of high demand
        - Standard (3 to 5 hours)
        - Bulk (5 to 12 hours)
    - <b>Glacier Deep Archive</b>
      - 99.99% availability
      - 2 flexible retrieval:
        - Standard (12 hours)
        - Bulk (48 hours)
      - Minimum storage duration of <b>180 days</b>
      - Lowest cost
- <b>Intelligent Tiering</b>
  - 99.9 availability
  - Move objects automatically between access tiers based on usage
  - Small monthly monitoring and auto-tiering fee
  - <b>No retrieval charges</b>

### Moving between Storage classes
![](images/storage_class_moving.png "Moving between storage class image")

### Lifecycle Rules
- Used to automate transition or expiration actions on S3 objects
- <b>Transition Action</b> (transitioned to another storage class)
- <b>Expiration Action</b> (delete objects after some time)
  - delete a version of an object
  - delete incomplete multi-part uploads
- Lifecycle Rules can be created for a prefix (ex: `s3://mybucket/mp3/*`) or objects tags (ex: Department: Finance)
> - When you apply a retention period to an object version explicitly, you specify a `Retain Until Date` for the object version
> - When you use bucket default settings, you don't specify a `Retain Until Date`. Instead, you specify a duration, for which every object
> version placed in the bucket should be protected.
> - Different versions of a single object can have different retention modes and periods

### S3 Analytics
- Provides analytics to determine when to transition data into different storage classes
- <b>Does not work for ONEZONE_IA & GLACIER</b>

### Performance
- 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD request per seconds per prefix
- Recommended to spread data across prefixes for maximum performance
- SSE-KMS may create a bottleneck in S3 Performance
- Performance Optimization
  - <b>Multi-part Upload</b>
    - Parallelize upload
    - <b>Recommended for files > 100MB</b>
    - <b>must use files > 5GB</b>
  - <b>Byte-range fetches</b>
    - Parallelize download requests by fetching specific byte ranges in each request
    - Better resilience in case of failures since we only refetch the failed byte range and not the whole file
  - <b>S3 Transfer Acceleration</b>
    - Speed up <b>upload and download</b> for large object (> 1GB) for global users
    - Data is ingested at the nearest edge location and is transferred over AWS private network (uses `CloudFront` internally)

### S3 Select
- Select a subset of data from S3 using <b>SQL queries (server-side filtering)</b>
- Less network cost
- Less CPU cost on the client side

### Data Transfer Cost
- Uploads to S3 are free
- Downloads from S3 are paid
- Using S3 Transfer Acceleration, you pay for transfer that are accelerated
- Since buckets are defined within a region, <b>data transfer within a region is free</b>

### S3 Notification Events
- Optional
- Generates events for operations performed on the bucket or objects
- Object name filtering using prefix and suffix matching
- Can create as many `S3 events` as desired
- Targets
  - SNS Topics
  - SQS Standard Queues (not FIFO Queues)
  - Lambda Functions
  - Amazon EventBridge

### Requester Pays Buckets
- Requester pays the cost of the request and the data downloaded from the bucket. The bucket owner only pays for the storage.
- Used to share large datasets with other AWS accounts
- The requester must be authenticated in AWS (cannot be anonymous)

### Object Lock
- WORM (Write Once Read Many) model
- Block an object version modification or deletion for a specified amount of time
- Modes:
  - <b>Governance mode</b>
    - Only users with special permissions can overwrite or delete the object version or alter its lock settings
  - <b>Compliance mode</b>
    - A protected object version cannot be overwritten or deleted by any user, including the root user
    - The object's retention mode can't be changed, and the retention period can't be shortened.
- Glacier Vault Lock
  - WORM (Write Once Read Many) model for Glacier
  - For compliance and data retention

### Access Points
- Each `Access Point` gets it own DNS policy to limit who can access it
  - A specific IAM user/group
  - One policy per Access Point -> Easier to manage complex bucket policies

### S3 Object Lambda
- Use AWS Lambda functions to change the object before it is retrieved by the caller application
- Only one `S3 bucket` is needed, on top which we create <b>S3 Access Point</b> and <b>S3 Object Lambda Access Points</b>

### S3 Batch Operations
- Can perform bulk operations on existing S3 Objects with a single request
  - Modify object metadata & properties
  - Copy objects between S3 bucket
  - <b>Encrypt un-encrypted objects</b>
  - Modify ACLs, tags
  - Restore object from S3 Glacier
  - Invoke Lambda function perform custom action on each object
- managges retries, tracks progress, send completion notification, generate reports
- <b>You can use S3 Inventory to get object list and use S3 Select to filter objects</b>

## Relational Database (RDS)
- Regional Service
- Supports Multi AZ
- AWS Managed SQL Database
- Supported Engines
  - Postgres
  - MySQL
  - MariaDB
  - Oracle
  - MSSQL
  - Aurora (AWS Managed Database)
- Backed by Elastic Compute Cloud `EC2` storage
- We don't have access to the underlying instance
- <b>DB Connection is made on port 3306</b>
- Security Groups are used for network security (must allow incming TCP traffic on port 3306 from specific IPs)

### Backups
- <b>Automated Backups</b> (enabled by default)
  - Daily full backup of the database (during the define maintenance window)
  - Backup retention: 7 days (max 35 days)
  - Transaction logs are backed-up every <b>5 minutes</b> (point in time recovery)
- <b>DB Snapshots</b>
  - Manually triggered
  - Backup retention: unlimited

### AutoScaling
- Automatically scales the RDS storage within the max limit
- Condition for automatic storage:
  - Free storage is less than 10% of allocated storage
  - Low-storage lasts at least 5 minutes
  - 6 hours have passed since last modification

### Read Replicas
- Allow us to scale the read operation (SELECT) on RDS
- Up to 5 read replicas (within AZ, cross AZ or cross region)
- Asynchronous Relication (seconds)
- Replicas can be promoted to their own DB
- Applications must update the connection string to leverage read replicas
- Network fee for repliation
  - Same region: free
  - Cross region: paid

>You can create a read replica as a Multi-AZ DB instance. A standby of the replica will be created in another AZ for 
> failover support for the replica.


### Multi AZ
- Increase availability of the RDS database by replicating it to another AZ
- Synchronous Replication
- Connection string does not require to be updated(both the database can be accessed by one DNS name, 
which allos for automatic DNS failover to standby database)
- When failing over, <b>RDS flips the CNAME</b> record for the DB instance to point at the standby,
which is in turn promoted to become the new primary
- <b>Cannot be used for scaling as the standby database cannot take read/write operation</b>.

### Encryption
- At rest encryption
  - KMS AES-256 encryption
  - Encrypted DB -> Encrypted Snapshots, Encrypted Replicas and Vice versa
- In flight encryption
  - SSL certificates
  - Force all connections to your DB instance to use SSL by setting the `rds.force_ssl` parameter to `true`
  - To enable encryption in transit, download the <b>AWS-provided root certificates</b> & used them when connecting to DB
To encrypt an un-encrypted RDS database:
  - Create a snapshot of the un-encrypted database
  - Copy the snapshot and enable encryption for the snapshot
  - Restore the database from the encrypted snapshot
  - Migrate applications to the new database, and delete the old database

### Access Management
- Username and Password can be used to login the database
- `EC2` instances & `Lambda` functions should access the DB using `IAM DB Authentication` (<b>AWSAuthenticationPlugin</b> with
<b>IAM</b>) - token based access
- EC2 instance or Lambda function has an IAM role which allows to make an API call to the RDS service to get <b>auth token</b>
which uses to access the MySQL Database.
- Only Works with MySQL and Postgres
- Auth token is valid for 15 mins
- Network traffic is encrypted in-flight using SSL
- <b>Central access management using IAM</b> (instead of doing it for each DB individually)

### RDS Events
- RDS event only provide operational events on the DB instance (not the data)
- To capture data modification, events, use <b>native function</b> or <b>store procedures</b> to invoke a `Lambda` function.


### Monitoring
- `CloudWatch` metrics for RDS
  - Gather metrics from the hypervisor of the DB instance
    - CPU utilization
    - Database Connections
    - Free-able Memory

- Enhanced Monitoring
  - Gathers metric from an agent running on the RDS instance
    - OS processes
    - RDS child processes
  - Used to monitor different <b>processes or threads on a DB instance</b> (ex. percentage of the CPU bandwidth and total memory
  consumed by each database process in your RDS instance)

### Maintenance & Upgrade
Any database engine level upgrade for an RDS DB instance will Multi-AZ deployment triggers both primary and standby DB instances
to be upgraded at the same time. This causes <b>downtime</b> until the upgrade is complete.
This is why it should be done during maintenance window.

## Aurora
- Regional Service (Supports global databases)
- Support Multi AZ
- AWS managed relational DB cluster
- Preferred over `Relational Database Service (RDS)`
- Auto-scaling (max: 128 TB)
- <b>Asynchronous Replication</b> (milliseconds)
- <b>Supports only MySQL & PostgreSQL</b>
- Cloud-optimized (5x performance improvement over MySQL on RDS, over 3x the performance of PostgreSQL on RDS)
- <b>Backtrack</b>: restore data at any point of time without taking backups

### Endpoints
- Writer Endpoint (Cluster Endpoint)
  - Always points to the master (can be used for read/write)
  - Each Aurora DB cluster has one cluster endpoint
- Reader Endpoint
  - Provides load-balancing for read replicas only (used to read only)
  - If the cluster has no read replica, it points to master (can be used to read/write)
  - Each Aurora DB cluster has one reader endpoint
- Custom Endpoint
  - Used to point to a subset of replicas
  - Provides load-balanced based on criteria other than the read-only or read-write capability of the DB instances
like class(ex, direct internal users to low-capacity instances and direct production traffic to high-capacity instances)

### High Availability & Read Scaling
- `Self-healing` (if some data is corrupted, it will be automatically healed)
- Storage is striped across 100s of volumes (more resilient)
- <b>Automated failover</b>
  - A read replica is promoted as the new master in less than 30 seconds
  - Aurora flips the <b>CNAME</b> record for your DB instance to point at the healthy replica
  - In case <b>no replica</b> is available, Aurora will attempt to <b>create a new DB instance</b> in the 
<b>same AZ</b> as the original instance. This replacement of the original instance is done on a <b>best effort basis</b> and may not succeed.
- Support for <b>Cross Region Replication</b>
- Aurora maintains 6 copies of your data across 3 AZ:
  - 4 copies out of 6 needed for writes (can still write if 1 AZ completely fails)
  - 3 copies out of 6 need for reads

> Each Read Replica is associated with a priority tier (0-15). In the event of a failover, Amazon Aurora will promote the Read Replica
> that has the highest priority (lowest tier). If two or more Aurora Replicas share the same tier, then Aurora promotes the replica that is 
> largest in size. If two or more Aurora Replicas share the same priority and size, then Aurora promotes an arbitary replica in the same promotion tier.

### Encryption & Network Security
- Encryption at rest using KMS (same as RDS)
- Encryption in flight using SSL (same as RDS)
- You can't SSH into Aurora instance (same as RDS)
- Network Security is managed using Security Groups (same as RDS)
- EC2 instances should access the DB using `IAM DB Authentication` but they can also do it using
credentials fetched from the `SSM Parameter Store` (same as RDS)

### Aurora Serverless
- Optional
- Automated database instantiation and auto scaling based on usage
- Good for unpredictable workloads
- No capacity planning needed
- Pay per second

### Aurora Multi-Master
- Optional
- Every node (replica) in the cluster can read and write
- Used for immediate failover for write node (high availability in terms of write). 
if disabled and the master node fails, need to promote a Read Replica as the new master (will take some time)
- <b>Client needs to have multiple DB connections for failover</b>

### Aurora Global Database
- Entire database is replicated across regions to recover from region failure
- Designed for <b>globally distributed application</b> with <b>low latency local reads</b> in each region
- 1 Primary Region (read/write)
- Up to 5 secondary (read-only) regions (replication lag < 1 second)
- Up to 16 Read Replicas per secondary region
- Helps for decreasing latency for clients in other geographical locations
- <b>RTO of less than 1 minute</b> (to promote another region as primary)

### Aurora Events
- Invoke a `Lambda` function from an Aurora MySQL-compatible DB cluster with a native function or a stored procedure (same as RDS)
- Used to capture data changes whenever a row is modified.

## Network

## Messaging

## Data Migration & Sync

## Access Management

## Distribution

## Monitoring & Audit

## Containerization

## Deployment

## Parameters 

## Encryption

## Analytics

## Security

## Disaster Recovery

## Extras