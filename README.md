[[Sviluppo]]

# AWS Developer Associate
N.B. In these notes i'll no add my previous knowledge from the [[AWS - Cloud Practitioner]]. So don't use this to study! You can study from a great course like this => https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/

To access to AWS you can use: 
- AWS Management Console
- CLI 
- SDK
Access Keys are secret, just like a password (Access Jey ID / Secret Access Key). Don't confuse them with key pairs to access via ssh to EC2.

Up to 5000 users per AWS account.
IAM policy simulator is a tool to help you understand, test and validate the effects of access control policies.


**IAM Credentials Report** => List all users and their credentials
**IAM Access Analyzer** => Identify the resources in your organization and accounts that are shared with an external entity.

To enable access from IAM users you can create an alias of your account. The alias cannot contain capital letters as a part of the alias name (because it will be a subdomain).


# EC2

Security groups only contain **allow** rules and are **stateful**l.

Dedicated Hosts: book a physical server and allows server-bound licenses
Dedicated Instances: no other customers will share your hardware

With Zonal Reserved Instances you can ave a capacity reservation for a given period of time in a specific AZ.

EC2 instances could be enabled for hibernation.

You can have **Elastic IP address**.
It is a static public IP address, you are charged if not used.




## EBS
**It's locked to an Availability Zone.**
Have a provisioned capaciti (size in GB and IOPS)

PIOPS => Optimized EBS volumes and optimized configuration stacks
PIOPS vs SSD, who is the faster? => SSD (because it is not a network storage)

EC2 instances can be launched from:
- A public AMI 
- Your own AMI 
- An AWS Marketplace AMI

### Volume Types
[Volume Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)
- gp2/gp3
	- IOPS and throughput can be increased independently
- io1/io2
	- io2 Block Express has superpowers!
	- Supports EBS Multi-attach
		- Multi-attach works in the same AZ
		- Just for clustered Linux applications
- st1
- sc1

![[Pasted image 20220205104444.png]]

## EFS
Works in multi-AZ
Really expensive => 3x gp2 (but it is pay per use and not for provisioned use)

# Load balancing & Auto scaling

A listener is a process that checks for connection requests. It is configured with a protocol and a port for front-end (client to load balancer) connections, and a protocol and a port for back-end (load balancer to back-end instance) connections.

Both the Load Balancer protocol and the instance protocol should be at the same layer. So for example you cannot use HTTPS on the LB protocol and SSL (layer 4) on the instance protocol.

Types of load balancers: 
- Classic LB
- **Application LB**
	- Support for redirects
	- Support for websocket
	- Routing to multiple target groups (by url, hostname, query string, headers)
	- Target groups:
		- EC2
		- Lambda
		- IP Addresses (private)
- **Network LB**
	- Target groups: 
		- EC2
		- IP Addresses (private)
		- ALB
- **Gateway LB**
	- Target groups: 
		- EC2
		- IP Addresses (private)

LB  could be *internal* or *external*

Sticky Sessions uses cookies (custom or application). You can also use duration-based cookies.

Cross-zone load balancing is enabled by default in ALB, could be enabled in NLB with additional fees. 

Clients can use SNI to specify the hostname they reach. With ALB and NLM you can add multiple SSL certificates.

With deregistration delay the LB stops to send new request to the EC2 instance to allow the old requests to be finished.

With ALB access logs you can capture detailed information about requests sent to your load balancer

### Auto Scaling Group
ASG groups cannot span across multiple regions.

To use custom metrics in ASG you have to: 
1. Send custom metric from application to CloudWatch (PutMetric)
3. Use the ClodWatch alarm as the scaling policy for ASG


Scaling poilcies: 
- Target tracking
- Simple/step
- Scheduled actions
- Predictive

In the cooldown period the ASW will not launch or terminate additional instances.

Backlog per instance metric => A metric calculated by the message and the average time of execution.

If there are the same number of instances in multiple AZs, and the ASG have to delete one instance, the instance will be randomly selected.

By default, ASG uses EC2 Status Checks, but if ELB Health Checks are enabled in ASD, both types are used.

Termination policies: 
- Default
	1. Determine which AZ has the most instances
	2. Determine which instance to terminate so as to align the remaining instances to the allocation strategy for the On-Demand or Spot Instance that is terminating and your current selection of instance types
	3. Determine whether any of the instances use the oldest launch template
	4. Determine whether any of the instances use the oldest launch configuration
	5. After applying all of the criteria in 2 through 4, if there are multiple unprotected instances to terminate, determine which instances are closest to the next billing hour



# DB
## RDS
Allows Point in time recovery every 5 minutes
Storage backed by EBS (gp2 or io1)

Backups have 7 days retention (up to 35)
You can create Snapshots.

If you have an automated backup and a single zone implementation, RDS get frozen momentarily during the backup.

Up to 5 read replicas, whitin AZ, cross AZ or Cross Region.
Replication is *async*

RDS Multi AZ has a **sync** replication. You can use it as a failover.
Multi-AZ is free.

Access Management => Can leverage IAM-based auth for MySQL and PostgreSQL, but you can also use traditional user&pass.
In case of IAM-auth the token has a lifetime of 15minutes.

Encryption has to be defined at launch time.

RDS db are usually deployed within a private subnet. You can access to the DB by leveraging security groups.

If no backup time is set, RDS assigns a random time period based on the region

Scaling compute will cause downtime
Read replicas are available for MySQL, PostgreSQL, MariaDB, Oracle and Aurora (not SQL
Server).


## Aurora
Up to 15 read replicas. Support for Cross Region Replication. The cool version of RDS.
Automatic failover in less than 30 seconds

## Elasticache
Managed Redis or Memcached
![[Pasted image 20220205111407.png]]

ElastiCache **doesn't support IAM**
You can use Redis AUTH or in case of Memcached support SASL auth.

In Elasticache you can have clusters.
If cluster disabled: 
- One primary node, up to 5 replicas
- Async replication
- Replicas are rea only
- all nodes have all the data
- Multi-AZ enabled by defauld

If cluster is enabled: 
- Data is partitioned across shards
- Each shard has a primary and up to 5 replica nodes
- Up to 500 nodes per cluster

Different Caching implementations: 
- Lazy loading / Cache-Aside / Lazy population
- Write Through

Cache evictions: 
- You delete the item
- Item is evicted cause of fully memory
- TTL

Elasticache can be used also to compute-intensive workloads to store intermediate results and avoid to recompute them.

Exam tip: the key use cases for ElastiCache are offloading reads from a Database, and storing the results of computations and session state. Also, remember that ElastiCache is an inmemory database and it’s a managed service

# Route53
0.50$ per month per hosted zone
Can set TTL for records

CNAME only for non root domain
Alias works for root and non root domain 

With calculated health checks you can monitor up to 256 child health checks


# S3
If uploading more than 5GB must use multi-part upload.

Encryption: 
- **SSE-S3**
	- Keys handled&managed by AWS
	- “x-amz-server-side-encryption": "AES256"
	- unique object keys
- **SSE-KMS**
	- Use KMS
	- leverages GenerateDataKey and Decrypt
	- “x-amz-server-side-encryption": ”aws:kms"
- **SSE-C**
	- Use your own key
	- key provisioned in each HTTP request (in headers)
	- HTTPS mandatory
- **Cliend side encryption**

MFA Delete => MFA is required to delete objects (could be enabled/disabled only from the root account)
Pre-Signed URL => URL valid only for a limited time. (can be generated for download and for upload)

Could enable Cross Region Replication and Same Region Replication. Copying is asynchronous. Only new objects are replicated and there is no chaining in replication.

Performances can be increased using Multi-part upload and S3 Transfer Acceleration.

With S3 Select & Glacier Select you can retrieve less data using SQL.

With **Event Notifications** you can add a listener to a put event.

### Athena
Used to perform analytics against S3 objects. Supports CSV, JSON, ORC, Avro and Parquet. $5.00 per TB of data scanned. 



# Developing on AWS
You can use --dry-run to simulate api calls (for example to check if I have the permissions to perform an api request).

If you get an error you can decode the error with "sts decode-authorization-message".

In the SDK the default region is us-east-1.


API Rate Limits !== Service Quotas
Service Qyotas can be increased by opening a ticket.

If an API fails usually we use the Exponential Backoff

API calls must be signed with Signature v4

# CloudFront
Origins: 
- S3 bucket
- Custom origin (HTTP)
	- ALB
	- EC2
	- Any HTTP backend

You can add a whitelist or a blacklist to restrict the access to some coutries.
You can invalidate part of the cache using **CreateInvalidation**

You have access to Signed URL (for a single file) or Signed Cookies (for multiple files).

Three price classes: 
1. Price class all
2. Price class 200
3. Price class 100

CloudFront uses origin groups (one primary and one secondary) to do failover

With Field Level Encryption you can add an additional layer of security the sensitive user information.

# ECS
We need a container management platform: 
- ECS
- Fargate
- EKS

A cluster is a logical grouping of EC2 instances that runs the ECS agent (load from a special AMI).

An ECS service helps define how many tasks should run and how they shoud be run.
You can also add a load balancer.

A task definition is a text file in JSON format that describes one or more containers, up to a maximum of 10.

**ECR** is a private Docker image repository. Access is controlled through IAM. 
**Fargate** is a serverless launch mode. You don't need to provision EC2 instances

With **task placement strategy** and **task placement constraints** you can define where to place a task.

Strategies: 
- Binpack (minimize the number of instances)
- Random
- Spread

You can mix the strategies.
Constraints: 
- distinctInstance (each task on a different instance)
- memberOf

You can share data between containers using Bind Mounts (for example sharing the /var/logs folder)

For ECS classic we must configure the /etc/ecs/ecs.config file

In the ECS port mapping you can set 0 as the host port to allow ECS to automatically assign an available port.

# Elastic beanstalk
Application => Versions => Environments

We could have web environments and worker environments

Deployment Modes: 
- Single instance
- Hight Availability with Load Balancer

Deployment options: 
- All at once
- Rolling
- Rolling with additional batches
- Immutable
- Blue/Green  (**not a direct feature of Beanstalk**)
- Traffic splitting (Canary Testing through Route53)

![[Pasted image 20220205133625.png]]

You can handle up to 1000 application versions. Old versions must be deleted using a lifecycle policy (based on time or space).

Extensions could be handled in the .ebextensions/ directory.
Under the hood, Beanstalk leverages Cloudformation.

Beanstalk in Single Docker Container doesn't use ECS.
In Multi Docker Container you have to add a Dockerrun.aws.json at the root of the source code used for the task definition.

Uses ZIP or WAR files

# CI/CD [[DevOps]]
![[Pasted image 20220205134354.png]]

### CodeCommit
Interactions are done using Git.
Authentication is done using SSH keys and HTTPS. Authorization is done from IAM policies.

Repositories are automatically encrypted using KMS

You can migrate code to codecommit from GIt repository in many wais like migrating all, cloning it, just some branches etc.

### CodeBuild
Leverages Docker under the hood. You can create your custom Docker image.
KSM for encryption of build artifacts.

Build instructions are in buildspec.yml

You can run CodeBuild locally leveraging the CodeBuild Agent (running on Docker).

By default it runs outside the VPC, you can run it inside your VPC.

There are some hooks (a set of instructions to do the deploy).

Configurations:
- One At a Time
- Half At A Time
- All At Once
- Custom

### CodeDeploy

In CodeDeploy you have to set an appspec.yml to specify deployment instructions!
All AWS Lambda and Amazon ECS deployments are blue/green. An EC2/On-Premises
deployment can be in-place or blue/green.

In CodeDeploy the sequence is Application stop, then download bundle > before install > install > after install > application start > validate service.

If a rollback happens, CodeDeploy redeploys the last known good revision as a **new deployment**.

**CodeStar** is an integrated solutions that groups all these tools, it is a free service.
**CodeArtifact** is used for artifact management
**CodeGuru** makes automated code reviews (reviewer & profiler)



# CloudFormation (IaaS)

Template components: 
- Resources
	- Over 224 resources
- Parameters
	- Provide inputs
	- Reference with Fn::Ref or simply !Ref
	- Pseudoparameters => Parameters enabled by default
	- Parameters are independent and cannot depend on each other
- Mappings
	- Variables
- Outputs
	- Values that can be passed to other stack as input
	- You can't delete the underlying stack until all the references are deleted 
	- Import with => !ImportValue (works only in the same region)
- Conditionals
- Metadata

Template helpers: 
- References 
- Functions

You can use nested stacks (stack as a part of other stacks)
StackSets => CRUD stacks across multiple accounts and region with a single operation

Drift => The current config is different from the CloudFormation definition


# AWS Monitoring, Troubleshooting & Audit

## CloudWatch
Retrieve statistics for the metric above => get-metric-statistics

### Metrics
A namespace is a container for CloudWatch metrics.

Create dashboard of metrics
Simple monitoring gets data every 5 minutes, but could enable detailed monitoring (gets data every minute)

### Logs
Log **groups**
Log **stream**

You can search and filter the log data coming into CloudWatch Logs by creating one or more *metric filters*

To add data through CLI => put-metric-data

You can export logs in S3 with **CreateExportTask**

To send custom data can use a CloudWatch Agent (on-premise too)
The new version is the CloudWatch Unified Agent 

To query on logs you can use filter expressions. 

Encryption (using KMS) is enabled at the log group level

### Alarms
Are used to trigger notifications for any metrics

### Events
Intercep events from AWS.
Events are passed to a target

EventBridge is the next evolution of CloudWatch Events. With EventBridge you can use third part event bus.


## X-Ray
**Tracing** is an e2e way to follow a request
Tracing is made of segments (and subsegments).

Instrumentation => Measure of product's performance, diagnose errors, and to write trace information

With **sampling** you can reduce the cost.
**Annotations** are key/value pairs used to index traces.
**Metadata** are key/value pairs not indexed

You can change the reservoir and rate.
The are some apis: 
- PutTraceSegments
- PutTelemetryRecords
- GetSamplingRules
- GetServiceGraph
- batchGetTraces
- GetTraceSummaries
- GetTraceGraph

To enable with Elastic Beanstalk you have just to add a line of code in the settings (not install the entire X-Ray agent).

With ECS you can use a Daemon (one X-Ray daemon per instance) or as a Side Car (one per Container)

ECS/EKS/Fargate => Create a Docker image that runs the daemon or use the official X-Ray Docker image

## CloudTrail
Management Events (default enabled)
Data events (default disabled, high volume operations)
Insights Events => Detect unusual activity

Events retention: 90 days (you can store on S3 and query with Athena)

# AWS Integration & Messagging
## SQS
I have studied SQS and SNS equivalents in the [[Programmazione ad oggetti distribuiti]] book in the PD course in UNISA.

Standard queue:
- default retention of messages 4 days (up to 14)
- Max 256kb per message sent

Poll SQS for messages (receive up to 10 messages at a time)
Amazon SQS uses either your Access Key ID or an X.509 certificate to authenticate your identity

**ChangeMessageVisibility**
After the **MaximumReceives** threshol is excedeed, the message goes into the DLQ
Delay a message => **DelaySeconds**
To enable long polling => **WaitTimeSeconds**
\> 256? Use the SQS Extended Client!

Useful API:
- CreateQueue, DeleteQueue
	- If you try to delete a queue with messages not consumed yet, the queue will be deleted with all the messages.
- PurgeQueue
- SendMessage, ReceiveMessage, DeleteMessage
- MaxNumberOfMessages
- ReceiveMessageWaitTimeSeconds
- ChangeMessageVisibility


Batch API are available for SendMessage, DeleteMessage, ChangeMessageVisibility

With FIFO you have de-duplication (content-based deduplication or explicit using an ID.

If there is no activity agains a queue for more than 30 consecutive days the queue may be deleted.


## SNS 
Up to 10M subscriptions per topic and 100k topics limit.

Can only have SQS FIFO queues as a subscriber for the FIFO Topic (for the Fanout pattern).
With the Fanout you can have filter policy.


## Kinesis
- Data streans => Stream
- Data Firehose => Load streams into AWS stores
- Data Analytics => analyze streams with SQL 

BIlling is per shard, retention between 1 day up to 365. Data can be reprocessed. Data cannot be deleted (immutable).

**PutRecord** API to add a record (1MB/sec or 1000records/sec per shard)

Consumers could be Classic (2MB/sec across all consumers) or Enhanced  (2MB/sec per consumer per shard).

KCL => Each shard could be readed by only one KCL instance


# Serverless stack 

## Lambda

From 128MB up to 10GB or RAM (increasing RAM will also improce CPU and network).
Timeout from 3 seconds up to 900 (15 mins)

Languages: 
- Custom Runtime API
- Container Image => Mutst implement che Lambda Runtime API 
You can test the containers locally using the Lambda Runtime Interface Emulator
Lambda Call Types: 
- Synchronous 
	- CLI, SDK, API Gateway, ALB
- Asynchronous
	- S3, SNS, CloudWatch Events, SQS, CodeCommit....
	- Can define a DLQ
	- Support for FIFO
	- Use an Event Source Mapping
		- Records need to be polled from the source
		- Lambda is invoked synchronously for SQS - Kinesis - DynamoDB
		- for other services such as Amazon S3 and SNS, the function is invoked asynchronously and the configuration is made on the source (S3/SNS) rather than Lambda.
		- Discarded events can go to a Destination
	- Could define Destinations

Lambda@Edge => deploy functions alongside the CloudFront CDN.
Can enable X-Ray tracing.

By default Lambda runs out of your VPC. To give resources access you have to define the VPC ID, Subnets and the Secutiry Groups. Lambda will create an ENI.

/tmp directory => Max size: 512MB
1000 concurrent executions, can set a "reserved concurrency".
Concurrency is managed by Application Auto Scaling

Lambda layers are used to run custom runtimes or to externalize dependencies.

Lambda functions in container => up to 10GB from ECR.
Versions are immutable

Aliases are pointers to versions and enable Blue/Green deployment. An alias cannot reference another alias.

CodeDeploy can help automate traffic shift for Lambda: 
- Linear
- Canary
- AllAtOnce

https://aws.amazon.com/premiumsupport/knowledge-center/lambda-iterator-age/


## DynamoDB

Primary key must be decided at creation time. Max item size => 400k.
Primary key: 
- Partition key
- Partition Key + Sort Key 

GUID => Global Unique Identifier

Provisioned mode: 
- WCU => items\*item_size (item_size gets rounded to the upper KB)
- RCU => one Strongly Consistend Read = items\*(item_size/4)

On demand is 2.5x more expensive

API:
- PutItem 
- UpdateItem
- GetItem
	- ProjectionExpression
- Query
	- KeyConditionExpression
	- FilterExpression
- Scan
	- Parallel Scan
- DeleteItem
- DeleteTable
- BatwhWriteItem
	- Up to 25 PutItem &&/|| DeleteItem
- BatchGetItem 
	- Up to 100 items

Can create up to 5 LSIs per table.
Local Secondary Index => Alternative Sort key (defined at table creation time)
Global Secondary Index => Speed-up queries on non-key attributes (can be added/removed after table creationg)

Optimistic Locking => Uses Conditional Writes
DAX => Cache for DynamoDB

DynamoDB Streams => Ordered stream of item-level modifications (retention up to 24 hours)

Transactions => all-or-nothing operations (consumes 2x WCU & RCU)

You can create on-demand backup capability to write backups on S3.

DynamoDB could communicate directly with an user using the API Gateway. You have to create integration request to allow the request to be forwarded to the DB.

Increases in throughput level of a table will typically take anywhere from a few minutes to a few hours.

Arrays are not supported

If your access pattern exceeds 3000 RCU or 1000 WCU for a single partition key
value, your requests might be throttled


## API Gateway

Stage variables are like env variables for API Gateway. Are passed to the "context" object in AWS lambda.

You can use a Canary Deployment for any stage. 

Integration types:
- MOCK 
- HTTP/AWS
	- Using mapping templates
		- Velocity Template Language
	- AWS_PROXY
	- HTTP_PROXY

Can enable cache (defined per stage, but can be override). To invalidate the cache use header: Cache-Control: max-age=0

You can define Usage Plan to allow your users to pay to use your API.
Account limit => 10k rps across all API. Can set Stage limit & Method limits.

Security: 
- IAM Permissions
- Lambda authorizer
- Cognito User Pools

Can use Websocket API. 

## SAM 
Transform Header indicates it's SAM template. 
san build | sam package | sam deploy

### SAR
Serverless Application Repository 


# CDK 
Define infrastructure in your preferred language

# Cognito
- User pools
	- DB of users
	- Triggers available
- Identity Pools
	- Get identities for "users" so they obtain temporary AWS credentials
- Cognito Sync (now AppSync)



![[Pasted image 20220206100534.png]]

## Other serverless 
### Step functions
Workflows as state machines. You can use retry or catch and can handle different types of errors.

Task => A single unit of work performed by a state machine

There are 2 types of workflows: 
- Standard
	- Over 2k/s execution but with max 1 year of duration
- Express
	- Max 5 min, over 100k/s executions

### AppSync

Managed service that uses GraphQL 

# Advanced Identity
## STS
Temporary access to AWS resources. Available as a global service. All regions are enabled by default but can be disabled.

Credentials will always work globally

API:
- AssumeROle
- AssumeRoleWithSAML
- AssumeRoleWithWebIdentity
- GetSessionToken
- GetFederationToken
- GetCallerIdentity
- DecodeAuthorizationMessage


## KMS
CMK Types: 
- Symmetric
- Asymmetric

API: 
- Encrypt
- Decrypt 
- GenerateDataKey
	- For Envelope Encryption
	- Enything over 4kb of data must use the Envelope Encryption

You can use the Data Key Caching

## SSM Parameter Store
Secure storage for configuration and secrets
Version tracking of configurations/secrets

## Secrets Manager
Capability to force rotation of secrets every X days 

# Other services
### SES
Send email lol

### AWS Certificate Manager
Provide TLS Certificates

### CloudMap
Resource discovery service

### Fault Injection Simulator (FIS)

# Testing
## [Examtopics](https://www.examtopics.com/exams/amazon/aws-certified-developer-associate/view)
- 131 - During an immutable environment update, the capacity of your environment doubles for a short time when the instances in the new Auto Scaling group start serving requests and before the original Auto Scaling group's instances are terminated. If your environment has many instances, or you have a low on-demand instance quota, ensure that you have enough capacity to perform an immutable environment update. If you are near the quota, consider using rolling updates instead.
- 141 - https://aws.amazon.com/premiumsupport/knowledge-center/iam-assume-role-cli/
- 147
	- Question: Users will be required to access AWS services and will be able to update their own passwords, according to an application being developed by a business.  
	Which of the following would allow the organization to manage users and authorization while allowing users to change their passwords on their own?
	- Answer: https://serverless-stack.com/chapters/cognito-user-pool-vs-identity-pool.html User pool -> To change password Identity pool -> To access AWS services
- 152
	- Question: An application has been created by a developer that publishes data to Amazon DynamoDB. Conditional writes have been enabled for the DynamoDB table. Writes are failing during high demand periods owing to a ConditionalCheckFailedException problem. 
	How can the developer improve the dependability of the program when numerous clients try to write to the same record?
	
	Answers:
	-   A. Write data to an Amazon SNS topic.
	-   B. Increase the amount of write capacity for the table to anticipate short-term spikes or bursts in write operations.
	-   C. Implement a caching solution, such as DynamoDB Accelerator or Amazon ElastiCache.
	-   D. Implement error retries and exponential backoff with jitter.
- 158 - Distraction error - missing one of two answers
- 179 
	- Question: A business builds, bundles, and packages its apps on-premises and stores them locally using a third-party technology. The firm runs its front-end apps on Amazon EC2 instances.  How does one deploy an application from the source control system to the EC2 instances?
	- Answer: Upload the bundle to an Amazon S3 bucket and specify the S3 location when doing a deployment using AWS CodeDeploy.
- 184:
	- Question: A corporation has automated their release pipelines using AWS CodePipeline. The development team is currently developing an AWS Lambda function that will deliver alerts when the status of each stage's action changes. How do I correlate the Lambda function with the event source?
	- Answer: 
		- CodeCommit/CodeDeploy/CodePipeline -> Notifications you can do from them to SNS only. 
		- Triggers you can do on CodeCommit to Lamda/SNS//CodeBuild schedule triggers. 
		- CodePipeline-> triggers from state changes from Lamda it is only Cloudwatch events.
- 185 - I'm still convinced my answer is correct
- 187 - English issue...
- 200
	- Question: Amazon S3 is structured as follows: S3:/BUCKET/FOLDERNAME/FILENAME.zip  Which S3 best practice would enhance speed when a single bucket receives thousands of PUT requests per second?
	- Answer: D is the answer, then and now. The practice of improving S3 performance has not changed by using different folder names as prefixes. If you had a bucket, with 10 x folders numbered 1-10, in which contents are places, that is 10x parallel streams of data. Hashing used to be the way to achieve this, but now sequential naming/numbering achieves the same result. You can still use hashes.
- 202
	- Question: A programmer is developing a REST API that will allow users to add goods to a shopping list. The service is developed on Amazon API Gateway and integrates with AWS Lambda. The shopping list items are sent to the function as query string arguments.  How should the developer transform query string parameters to Lambda function arguments?
	- Answer: You must use mapping templates by yourself, you can also use the integration type but in this case it is asking you to make the mapping
- 208 - cache invalidation problem... write through instead of invalidate cache and update DB
- 219
	- Question: A program may have hundreds of users. Each user may access the application through various devices. The Developer want to give these users unique IDs regardless of the device they are using.  Which mechanism should be utilized to generate unique identifiers?
	- Answer: Whenever you see any question with keywords mobile and/or web authentication/authorization, if the Cognito is one of the given options, it has to be the correct one!
	- https://docs.aws.amazon.com/cognito/latest/developerguide/developer-authenticated-identities.html
- 225 => Cache session data, minimum latency => ElastiCache
- 226
	- Question: Sensitive data such as medical records, medical imaging, bank statements, and invoices are uploaded to Amazon S3 as part of the program. All papers must be sent and kept securely. All access to documents must be documented for auditing purposes. Which technique is the MOST SECURE?
	- Answer: I would go with (D) because the goal of the question is security related. Take a look at S3 FAQ: https://aws.amazon.com/s3/faqs/?nc1=h_ls. In the Security topic, it is possible to see that encryption with SSE-KMS provides improved auditing compared to other options.
- 235
	- To increase read speed, an application makes use of a single-node Amazon ElastiCache for Redis instance. Demand for the application has grown significantly over time, putting an increasing strain on the ElastiCache instance. It is vital that this cache layer is capable of **handling the load and being robust in the event of a node failure**. What can be done by the developer to meet load and resilience requirements?
	- https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Scaling.RedisReplGrps.ScaleOut.htm
- 243 - Distraction error => MFA could be used in Cognito too
- 249
- 253
	- Which of the following setups enables customers to be notified quietly whenever an update is ready for their other devices?
	- Amazon Cognito uses the Amazon Simple Notification Service (SNS) to send silent push notifications to devices. A silent push notification is a push message that is received by your application on a user's device that will not be seen by the user.

## Insights from wrong questions


- If your environment has many instances, or you have a low on-demand instance quota, ensure that you have enough capacity to perform an immutable environment update. If you are near the quota, consider using rolling updates instead.
-  https://aws.amazon.com/premiumsupport/knowledge-center/iam-assume-role-cli/
-  https://serverless-stack.com/chapters/cognito-user-pool-vs-identity-pool.html User pool -> To change password Identity pool -> To access AWS services
- When you build on-premise and you want to deploy on cloud, you have to upload the bundle to an Amazon S3 bucket and specify the S3 location when doing a deployment using AWS CodeDeploy.
- CodeCommit/CodeDeploy/CodePipeline -> Notifications you can do from them to **SNS** only. 
		- Triggers you can do on CodeCommit to Lamda/SNS/CodeBuild schedule triggers. 
		- CodePipeline-> triggers from state changes from Lamda it is only **Cloudwatch events**.
- Improving bucket performances =>  If you had a bucket, with 10 x folders numbered 1-10, in which contents are places, that is 10x parallel streams of data. Hashing used to be the way to achieve this, but now sequential naming/numbering achieves the same result. You can still use hashes.
- Whenever you see any question with keywords mobile and/or web authentication/authorization, if the Cognito is one of the given options, it has to be the correct one!
- In S3 encryption, encryption with SSE-KMS provides improved auditing 
- In Redis in single-node instance you can achieve scaling and resiliency using read replicas.
- Amazon Cognito uses the Amazon Simple Notification Service (SNS) to send silent push notifications to devices. A silent push notification is a push message that is received by your application on a user's device that will not be seen by the user.
