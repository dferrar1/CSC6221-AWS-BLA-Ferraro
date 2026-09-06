# CSC6221-AWS-BLA-Ferraro
Solutions Architect associate BLA

# BLA 01: AWS Application Integration, Containers & Serverless

A recorded technical presentation covering the application integration, container, and
serverless domains of the **AWS Certified Solutions Architect – Associate (SAA-C03)**
curriculum — delivered as three segments of roughly seven minutes each.

| | |
|---|---|
| **Author** | Dylan Ferraro |
| **Course** | CSC6221 |
| **Deliverable** | recorded presentation + slide deck |
| **Artifacts** | [SAA BLA #1.pptx](https://github.com/user-attachments/files/31872508/SAA.BLA.1.pptx) | https://github.com/user-attachments/assets/d1f8f6f3-971f-4cda-91d4-4a8be0939afa |

---

## Purpose of Activity

The goal was to take a large, loosely connected body of study notes and force it into a
form that requires actual understanding: **a timed, recorded explanation aimed at another
person.**

Notes let you get away with knowing *that* SQS has a visibility timeout. A presentation
does not — you have to explain *why* it exists, what breaks without it, and what a
practitioner should set it to. That gap is the point of the exercise.

Three specific objectives:

1. **Consolidate three separate SAA-C03 domains** — messaging/integration, containers, and
   serverless — into one coherent architecture narrative rather than three disconnected
   service lists.
2. **Move from recall to decision-making.** For every service pair that gets confused on
   the exam (SQS vs SNS vs Kinesis, Data Streams vs Firehose, ECS vs EKS, CloudFront
   Functions vs Lambda@Edge), produce a decision rule rather than a definition.
3. **Practice technical communication under a time constraint.** Twenty-two minutes for
   roughly twenty services means every slide has to earn its place.

The deck is built around **one reference architecture** — an order-processing pipeline —
that every service is then plugged into, so the services are introduced as answers to
problems rather than as vocabulary.

---

## Technologies Used

### AWS Services

**Application Integration & Messaging**
- Amazon SQS (Standard and FIFO queues, dead-letter queues, visibility timeout, long polling)
- Amazon SNS (topics, subscriptions, FIFO topics, message filtering, fan-out)
- Amazon Kinesis Data Streams (shards, partition keys, provisioned & on-demand capacity)
- Amazon Data Firehose (buffered delivery, format conversion, Lambda transformation)
- Amazon MQ (managed ActiveMQ / RabbitMQ, multi-AZ failover)
- Amazon EventBridge (event-driven task invocation, scheduling)

**Containers**
- Amazon ECS (EC2 and Fargate launch types, task definitions, service auto scaling, capacity providers)
- AWS Fargate (serverless container compute)
- Amazon ECR (private registry, image vulnerability scanning)
- Amazon EKS (managed node groups, self-managed nodes, Fargate profiles, CSI drivers)

**Serverless & Compute**
- AWS Lambda (concurrency models, provisioned concurrency, SnapStart, VPC integration, container images)
- AWS Step Functions (workflow orchestration)
- CloudFront Functions and Lambda@Edge (edge compute)
- Amazon API Gateway (REST, edge-optimized / regional / private endpoints, authorizers)

**Data & Storage**
- Amazon DynamoDB (capacity modes, DAX, Streams, Global Tables, TTL, PITR, S3 import/export)
- Amazon S3 (event notifications, data lake destination)
- Amazon EFS (shared container storage across AZs)
- Amazon RDS / Aurora (RDS Proxy, event notifications, Lambda invocation from the database)

**Security, Identity & Operations**
- AWS IAM (identity policies, instance profiles, ECS task roles)
- Resource-based policies (SQS access policies, SNS access policies, API Gateway resource policies)
- Amazon Cognito (User Pools, Identity Pools / federated identity)
- AWS KMS (encryption at rest)
- AWS Secrets Manager / SSM Parameter Store
- AWS Certificate Manager
- Amazon CloudWatch (metrics, alarms, logs) and Application Auto Scaling
- Elastic Load Balancing (ALB, NLB)
- Amazon VPC (interface endpoints, subnets, security groups, ENIs)

### Tools, Languages & Technologies


| **Containerization** | Docker, Dockerfiles, Kubernetes, container registries |
| **Databases** | DynamoDB (NoSQL / key-value), Amazon RDS & Aurora (relational), Amazon Redshift & OpenSearch (analytics targets) |
| **Languages & runtimes** | Python, Node.js, JavaScript (CloudFront Functions), Java, C#, Ruby *(Lambda-supported runtimes discussed)* |
| **Protocols & standards** | HTTPS/TLS, REST, WebSocket, JMS, AMQP, MQTT, STOMP, OpenWire, SAML |
| **Data formats** | JSON, CSV, Parquet, Avro, DynamoDB JSON, ION |
| **SDKs & libraries** | AWS SDK, Kinesis Producer Library (KPL), Kinesis Client Library (KCL) |


---

## Concepts Learned

### Coupling is the root problem, and buffering is the general solution

Synchronous service-to-service calls propagate both **load** and **failure** backwards
into healthy components. Inserting a buffer between producer and consumer decouples them
on three separate axes at once: availability (the receiver can be down), scaling (the two
sides scale on different curves), and throughput (a spike becomes a longer queue rather
than dropped requests).

### The three messaging primitives differ by *delivery semantics*, not by scale

This turned out to be the most clarifying idea in the whole domain:

| Primitive | Who receives a message | Persistence | Mental model |
|---|---|---|---|
| **SQS** | Exactly one consumer | Until deleted (max 14 days) | A shared to-do list |
| **SNS** | Every subscriber | None — push and forget | A broadcast |
| **Kinesis** | Every consumer, independently paced | Full retention window (up to 1 year) | An append-only log |

Choosing correctly means asking "who needs to see this, and does it need to survive?" —
not "how much traffic do I have?"

### Fan-out (SNS → SQS) composes the strengths of both

SNS alone is fire-and-forget; a subscriber that is down loses the message. Putting SQS
queues behind an SNS topic yields broadcast delivery *plus* per-consumer durability,
retries, and independent processing rates — and lets you add subscribers later without
touching the producer. It also works around the S3 constraint of one event rule per
event-type-and-prefix combination.

### At-least-once delivery makes idempotency a design requirement

SQS standard queues can deliver duplicates and can deliver out of order. The visibility
timeout is the mechanism behind this: a message reappears if it isn't deleted in time.
The correct response is usually **not** to reach for FIFO — it's to make consumers
idempotent, and reserve FIFO for cases where ordering is genuinely part of the business
logic (at the cost of dropping from unlimited throughput to 300 msg/s, or 3,000 batched).

### Compute is a responsibility spectrum, not a set of alternatives

EC2 → ECS on EC2 → Fargate → Lambda is a continuous trade of **control** for **reduced
operational surface area**. Framing it as a responsibility matrix — who owns the OS, the
orchestration, the scaling, the code — makes the selection criterion obvious and removes
the temptation to treat "more serverless" as automatically better.

### Two IAM roles in ECS, doing two different jobs

The **EC2 instance profile** is what the ECS *agent* uses (registering with the cluster,
pulling from ECR, shipping logs to CloudWatch). The **ECS task role** is what the
*application inside the container* uses. Conflating them is both a common exam distractor
and a real source of over-privileged tasks.

### Identity policies and resource policies are two halves of one decision

IAM policies answer "what may this principal call?" Resource policies — the SQS access
policy, the SNS access policy, API Gateway resource policies — answer "who may act on this
resource?" Cross-account access and service-to-service publishing require the resource
side, which is why a correctly-IAM'd fan-out subscriber can still receive nothing.

### Concurrency, not CPU, is the scaling unit in serverless

Lambda's default 1,000 concurrent executions is an account-and-region ceiling shared by
every function. Reserved concurrency partitions it; provisioned concurrency and SnapStart
attack cold-start latency; and the same shift in thinking applies to queue consumers,
which should scale on **backlog depth** (`ApproximateNumberOfMessagesVisible`) rather than
on CPU utilization.

### Managed does not mean unbounded

Amazon MQ is fully managed but runs on brokers, so it does not scale like SQS. Firehose is
fully managed but buffers, so it is near-real-time and cannot replay. Fargate removes
instance management but not task sizing. Every managed service trades away a specific
capability, and knowing *which one* is what makes the selection defensible.

---

## Lessons Learned

**Explaining a service out loud exposes gaps that re-reading notes does not.**
Several topics I would have marked as "known" — the difference between reserved and
provisioned concurrency, why fan-out uses queues instead of subscribing Lambda directly —
fell apart the first time I tried to say them in a sentence. Writing the narration was a
more effective diagnostic than any practice question.

**Errors survive in notes until you have to defend them.**
Verifying every number against AWS documentation before recording surfaced several
mistakes I had carried for weeks — most notably an SQS message size limit recorded as
1024 KB (the actual limit is **256 KB**) and a Kinesis record size recorded as 10 MiB (the
actual limit is **1 MB** per record). Both would have been wrong in front of an audience.
Facts you never state out loud never get checked.

**A single worked example beats a service catalog.**
The first outline of this deck was organized service by service and read like an index.
Rebuilding it around one order-processing pipeline — and introducing each service as the
answer to a problem in that pipeline — made both the delivery and the retention
dramatically better. It also naturally surfaced the integration points, which are exactly
what the exam tests.

**Time pressure is a useful editor.**
A seven-minute segment holds roughly ten slides. That constraint forced a real decision on
every topic: is this a *decision rule* someone can act on, or is it trivia? Most of what
got cut was trivia. The constraint improved the content rather than limiting it.

**Failure modes are the most transferable knowledge.**
The troubleshooting slides were the hardest to write and are the most valuable part of the
deck. Knowing that visibility timeout exists is study; knowing that *"my message processed
twice"* almost always means the timeout is shorter than the processing time is
operational. Organizing that section as **symptom → cause → fix** made the underlying
mechanisms easier to remember than definitions ever did.

**Recording in segments rather than in one pass.**
Attempting a single 22-minute take meant every mistake cost the whole run. Recording one
segment at a time, with the segment divider slides as natural cut points, made retakes
cheap and the final result noticeably more composed.



## References

- [AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide](https://aws.amazon.com/certification/certified-solutions-architect-associate/)
- [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/)
- [Amazon SNS Developer Guide](https://docs.aws.amazon.com/sns/latest/dg/)
- [Amazon Kinesis Data Streams Developer Guide](https://docs.aws.amazon.com/streams/latest/dev/)
- [Amazon ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
