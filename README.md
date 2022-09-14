# AWS-Solutions-Architect-Associate (C00-003)
___

## Computing

### Elastic Compute Cloud (EC2)
- Regional Service
- EC2 (Elastic Compote Cloud) is an Infrastructure as a Service
- Stop & Start of an instance may change its public IP but not its private IP
- <b>AWS Compute Optimizer</b> recommends optimal AWS compute resource for your workloads
- There is a vCPU-based On-demand instance soft limit per region

### EC2 User Data
- It is possible to bootstrap our instances using an EC2 User data script
- bootstrapping means launching commands when a machine starts
- Scripts is only run once at the instance first start
- EC2 user data is used to automate boot task such ass: 
  - Installing updates
  - Installing software
  - Downloading commons files from the internet
- EC2 User data scripts runs with root user privilege

### Instance Types
- General purpose
  - Great for diversity of workloads such ass web servers or code repositories
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
  - Server hard is allocated to a specific company
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
    - It becomes acive after the spot instance is interrupted.
    - If you stop the spot instance, the requet will become active only afer you start the spot instance.
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
## Elastic IP
- Static public IP that you own as long as you don't delete it
- Can be attached to an EC2 Instance(even when it is stopped)
- Soft limit of 5 elastic IPs per account
- Doesn't incur charges as long as the following conditions are met
  - is associated with an Amazon EC2 instance
  - instance associated with the elastic IP is running
  - the instance has only one Elastic IP attached to it

## Placement Groups
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
- A primary ENI is created and attached to the instance upon creation and will be deleted automatically upon instance terminatio.
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
  - EBS root volumne is preserved
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

___

## Storage

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