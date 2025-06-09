# Amazon-DynamoDB
Amazon DynamoDB is a fully managed, serverless, NoSQL database service provided by AWS, designed to deliver high performance, scalability, and reliability for applications requiring fast and predictable access to data. It supports both key-value and document data models, making it highly flexible for handling structured, semi-structured, or unstructured data.
- Below are the key concepts and features of DynamoDB, ideal for documentation and understanding its capabilities.

## Key Concepts of DynamoDB
### Serverless
- DynamoDB requires no server management. AWS handles provisioning, patching, and scaling, with pay-as-you-go pricing. It scales automatically to meet demand and can scale to zero without cold starts.

### NoSQL Database
- Supports key-value and document data models with a flexible, schema-less design. It does not support JOIN operations, encouraging denormalization, but offers strong read consistency and ACID transactions.

### Fully Managed
- AWS manages setup, configuration, maintenance, high availability, hardware provisioning, security, backups, and monitoring, allowing developers to focus on application development.

### Performance
- Delivers single-digit millisecond latency at any scale, optimized for high-performance workloads. It can handle over 10 trillion requests per day and peaks exceeding 20 million requests per second.

### Scalability 
- Scales horizontally by automatically distributing data across multiple servers, supporting virtually unlimited storage and throughput. It offers two capacity modes: on-demand (automatic scaling) and provisioned (manual capacity settings).

## Use Cases
**Ideal for applications requiring high scalability, such as:**
- Financial services (e.g., live trading, loan management, transaction ledgers)
- Gaming (e.g., game state, leaderboards)
- Streaming (e.g., metadata indexing, recommendations)
- IoT (e.g., real-time data processing)
- Mobile apps and content management systems

### Capabilities:
- _**Global Tables:**_ Enables multi-active replication across AWS regions for low-latency access and 99.999% availability.
- **_ACID Transactions:_** Supports atomic, consistent, isolated, and durable transactions across one or more tables, suitable for financial transactions or order processing.
- _**Change Data Capture (CDC)**:_ Uses DynamoDB Streams and Amazon Kinesis to capture data changes for real-time processing.
- _**Secondary Indexes**:_ Global and local secondary indexes enable flexible querying without impacting performance.

### Security:
- Integrates with AWS Identity and Access Management (IAM) for fine-grained access control.
- Encrypts data at rest using AWS Key Management Service (KMS), with options for AWS-owned keys (free) or customer-managed keys (fee-based).
- Complies with standards like HIPAA, PCI DSS, and GDPR.

### Resilience:
- Replicates data across three Availability Zones for 99.99% availability.
- Offers continuous backups with point-in-time recovery (up to 35 days) and on-demand backups for additional protection.

### Pricing:
- Charges based on read capacity units (RCUs), write capacity units (WCUs), and storage.
- On-demand mode charges per operation; provisioned mode allows predefined throughput.
- Free tier includes 25 GB of storage, 25 WCUs, and 25 RCUs, supporting up to 200 million requests per month.

### Service Integrations:
- Integrates with AWS Lambda for serverless compute (e.g., triggers on data changes).
- Supports data import/export with Amazon S3.
- Enables zero-ETL integration with Amazon Redshift and OpenSearch for analytics.
- DynamoDB Accelerator (DAX) provides caching for up to 10x read performance improvement.
- _**Access Methods:**_ Supports AWS Management Console, AWS CLI, NoSQL Workbench, and DynamoDB APIs for interaction.

---

## Comparison Between DynamoDB and RDS
Amazon RDS (Relational Database Service) is a managed relational database service supporting SQL databases like MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora. While both DynamoDB and RDS are fully managed by AWS, they cater to different needs based on data models, scalability, and use cases. The table below compares their key features for clear documentation.

























