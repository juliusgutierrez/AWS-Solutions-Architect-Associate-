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
- Can be sued to modify `CloudFront` request & responses.
- We can create a global application using `Lambda@Edge` where `S3` hosts a static website
which uses client side JS to ssend requests to CF which will process the request in a lambda function
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
### Instance Store
### Elastic Block Storage (EBS)
### Elastic File System (EFS)
### Simple Storage Service (S3)
### Relational Database (RDS)

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