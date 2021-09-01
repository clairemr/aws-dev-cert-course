## EC2
#### SSH into instances
chmod 0400 key pair file to change permissions, then
`ssh -i ec2-instances.pem ec2-user@{insert public ipv4 address}`
- from inside folder containing that certificate
- disconnect from vpn to access

#### IAM
don't use aws configure with EC2 instances - may expose your credentials to others accessing the instance. Instead add IAM role
from security tab (permissions = IAMReadOnlyAccess), then you'll be able to use commands like aws iam list-users

#### ASGs
read https://aws.amazon.com/blogs/aws/new-application-load-balancer-sni/

#### Route 53
DNS is a collection of rules and records which helps clients understand how to reach a server through its domain name. Most common records in AWS:
- A: hostname to IPv4
- AAAA: hostname to IPv6
- CNAME: hostname to hostname, only for non root domain e.g. something.domain.com
- Alias: hostname to AWS resource, both root & non root e.g. domain.com works. Free  + include native health check
Where do the front door CIDR ranges come from? Can't see them in devops main.ts or in the front door cdk
^follow up, where does this all get set? cdk or manually?

#### SDK, CLI etc
- 125 sdk: if you don't specify or configure a default resion, then us-east-1 will be chosen by default
- 126 if you get ThrottlingException intermittently, use exponential backoff. Only implement retries on 5xx server errors & throttling, not on 4xx (means something sent wrong, will keep getting same errors)
- 127 look at credentials options

#### S3
- 137 review Different S3 storage types

#### ECS
164 ecs placement
- binpack: fill up containers first
- spread: spread across AZs
- random
Also review same lecture task placement constraints
165 auto scaling options e.g. target tracking, step caling & scheduled scaling ~1min
168 ecs summary and exam tips

#### Metrics
Metric is a variable to monitor (belonging to namespaces e.g. EC@, custom metric)
Dimension is an attribute of a metric (e.g. instance id), up to 10 per metrics

#### SQS, SNS & Kinesis
SQS scales automatically
SQS messages limited to 256mb, unless using extended client library (s3 bucket), allows up to 2GB
Data streams real time, ~200ms
Firehose is near real time, because data written in batches
Kinesis Data Streams vs SQS FIFO: FIFO best for dynamic number of consumers - e.g. could have one per group ID, whereas Data Streams limited to 5 consumers in parallel. Kinesis better if you have heaps of data, can receive up to 5 MB/s of data with ordering on shards
Which SQS FIFO message attribute allows two messages to be processed in order? MessageGroupId

#### Lambda
Make sure function & s3 bucket in same region to use them
280 error handling, around 3mins
recommended to set sqs visibilty timeout to 6x lambda function timeout
When you use an event source mapping to invoke your function, Lambda uses the execution role to read event data. Best practise is to create one lambda execution role per function
every lambda must have an iam role. basic role created automatically through console to allow writing to logs
also has resource based policies to allow other aws services to invoke the lambda function, e.g. s3
import os to access env variables in lambda. use as return os.getenv("VARIABLE_NAME")
by default lambda launched outside your own VPC (in aws vpc) and therefore can't access resources in your VPC (e.g. RDS, Elasticache, internal ELB). can however access any public website & dynamoDB (things outside vpc)
^to solve, deploy in vpc
292: initialise outside handler (e.g. db connect) so execution context can re-use for multiple function invocations

#### DynamoDB
One _write capacity unit_ represents one write per second for an item up to 1kb in size. Larger items = more WCU consumed
One _read capacity uni_ represents one strongly consistent read/second or two eventually consistent reads/second up to 4kb. Larger = more RCU consumed
Both round up e.g. 1.5kb counts as 2kb for wcu (2 wcu needed), 6kb = 2rcu needed. Both spread evenly between partitions
313 all dynamodb apis
315 GSI vs LSI
321 ~2mins calculate transactional capacity
322 comparison of options for session state
324 write types

#### API Gateway
To invoke specific lambda version, use functionName:${stageVariables.lambdaAlias} in the Lambda Function input field. Then need to add permission to allow api gateway to invoke lambda function, changing stageVariable to alias name e.g. DEV
Review and try hands on in 342 CORS

#### SAM
Any time you see a transform, you know it's a SAM template
e.g. in YAML Transform: 'AWS::Serverless-2016-10-31'. 
See 351 creating first SAM Project for YAML details

# Review
### Rewatch
- Lecture 34 (ec2 instance type basics) & 44 (instance launch types) pre exam to revise different EC2 instance types
- Lecture 52 EBS Volume Types, 56 (EFS vs EBS) for summary
- Lecture 59 (5ish mins in) types of load balancers, 7:30ish errors
- Most of RDS section, didn't do well on that quiz...
- Lecture 104, three tier architecture
- Lecture 112, S3 Security & Bucket Policies
- 217 Cloudwatch Metrics (types)
- 234 CloudTrail vs CloudWatch vs Xray
- 260, 3:50 Kinesis Data strems vs Fireho
- 263 SQS vs SNS vs Kinesis
- Lecture 269, lambda & alb hands on, try to build something 
- 280 lambda event source mapping, lots of details to review
- 306 lambda limits. Revise event source mapping as well
- 349 SAM overview (probs won't need anything else from this section) + 358 SAM summary
- 369 cognito differences
- 375 step functions standard vs express
- 378 STS
- 379 advanced IAM: iam + bucket policies combined, dynamic policies e.g. ${aws:username}, inline vs managed policies. Actually just review this whole section

- When to use ELB vs ALB

- (Digital Cloud aws resources)[https://digitalcloud.training/certification-training/aws-developer-associate/]

#### Cosmos questions
- does cosmos allow for putting things in multiple availability zones?/regions




cntlm conf lives at /usr/local/etc/cntlm.conf

https://www.infoq.com/articles/aws-vpc-explained/ <-- good summary of vpcs and subnets

Security groups vs IAM
https://stackoverflow.com/a/42033566 
IAM roles are for restricting AWS user/account/role access to the AWS API.
Security groups are for restricting network access to resources that exist inside your VPC.
Note how EC2 and RDS (and Redshift and Elasticache...) are servers that exist in your VPC, and you interact with those resources by making direct network connections to those servers. So you secure network access to these with Security Groups.

Note how you have no visibility into the what servers your S3 (or DynamoDB or SQS or SNS...) resources are on, those resources are not running inside your VPC, and you interact with those resources exclusively via the AWS API. So you secure AWS API access to these via AWS Identity and Access Management (IAM).


export AWS_PROFILE=claire