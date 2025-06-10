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

<details>
  <summary>Comparison Table Between DynamoDB and RDS</summary>

### DynamoDB vs Amazon RDS – Feature Comparison

| **Feature**      | **DynamoDB**                                                                    | **Amazon RDS**                                                                                       |
| ---------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Type**         | NoSQL, fully managed                                                            | Relational (SQL), supports multiple engines (Aurora, MySQL, MariaDB, PostgreSQL, Oracle, SQL Server) |
| **Data Model**   | Key-value and document                                                          | Relational, with tables, rows, and columns                                                           |
| **Schema**       | Flexible, schema-less                                                           | Predefined schema                                                                                    |
| **Scalability**  | Horizontal scaling, virtually unlimited storage and throughput                  | Vertical scaling, or horizontal with read replicas                                                   |
| **Performance**  | Single-digit millisecond latency, optimized for high-throughput NoSQL workloads | Optimized for relational queries and transactions, up to 40,000 IOPS (Aurora)                        |
| **Use Cases**    | Real-time bidding, shopping carts, mobile apps, gaming, IoT, unstructured data  | Enterprise apps, CRM, e-commerce, data warehouses, traditional apps                                  |
| **Availability** | Replicated across 3 Availability Zones, 99.99% SLA; global tables (99.999%)     | Multi-AZ deployments optional, active-passive failover, 99.9% SLA                                    |
| **Security**     | IAM integration, AWS KMS encryption (AWS-owned key free)                        | IAM for MySQL/PostgreSQL, AWS KMS encryption, SSL support                                            |
| **Backups**      | Point-in-time recovery (PITR) up to 35 days, on-demand backups                  | PITR via automatic backups, snapshots stored in Amazon S3                                            |
| **Storage Size** | Virtually unlimited                                                             | Aurora: 128 TB, MySQL/MariaDB/Oracle/PostgreSQL: 64 TB, SQL Server: 16 TB                            |
| **Querying**     | Fast key-based access, secondary indexes, limited join support                  | Complex SQL queries, including joins and advanced data manipulations                                 |
| **Pricing**      | Pay for reads (4KB), writes (1KB), storage; on-demand/provisioned modes         | Monthly fee per instance, pay-as-you-go, reserved instances cheaper                                  |
| **Consistency**  | Eventual or strong consistency options                                          | Strong consistency (ACID transactions)                                                               |
| **Maintenance**  | Fully managed, no manual intervention needed                                    | Fully managed, but some engine-specific configurations required                                      |

---

### Detailed Comparison Insights
- **Data Model and Schema**
  - **DynamoDB:** Its schema-less design allows for dynamic data structures, making it ideal for applications with evolving requirements, such as mobile apps or IoT. Data is stored as key-value pairs or JSON-like documents.
  - **RDS:** Requires a predefined schema, enforcing structured data with tables, rows, and columns. This is suitable for applications needing relational integrity, like financial systems.
    
- **Scalability**
  - **DynamoDB:** Scales horizontally by automatically partitioning data across servers, supporting massive datasets and high request rates (e.g., over 20 million requests per second).
  - **RDS:** Primarily scales vertically by upgrading instance types. Horizontal scaling is possible with read replicas, but it’s less seamless than DynamoDB’s approach.

- **Performance**
  - **DynamoDB:** Optimized for NoSQL workloads, delivering low-latency access for key-based queries. DAX caching can reduce read times from milliseconds to microseconds.
  - **RDS:** Excels in relational workloads, supporting complex SQL queries and transactions. Performance depends on the database engine and instance type (e.g., Aurora offers up to 40,000 IOPS).

- **Use Cases**
  - **DynamoDB:** Best for high-traffic, unstructured data applications like real-time bidding, gaming, or IoT. It powers Amazon’s Prime Day and Alexa, handling over 1 billion requests per hour.
  - **RDS:** Suited for structured data applications requiring complex queries, such as CRM, e-commerce, or data warehousing.
    
- **Availability and Resilience**
  - **DynamoDB:** 3 AZ replication by default, Automatically replicates data across three Availability Zones, with global tables offering multi-region replication for higher availability.
  - **RDS:**  Multi-AZ deployments are optional, using active-passive failover, which may incur additional costs and complexity.

- **Cost**
  - **DynamoDB:** Charges based on read/write operations and storage, with on-demand pricing offering flexibility but potentially higher costs for unpredictable workloads.
  - **RDS:** Charges per instance, with reserved instances offering cost savings for predictable workloads. Storage and I/O operations add to the cost.

---

### Choosing Between DynamoDB and RDS
- **_Choose DynamoDB when:_**
  - Your application handles unstructured or semi-structured data.
  - You need high scalability and low-latency performance for large-scale workloads.
  - Serverless architecture and minimal operational overhead are priorities.
  - Examples: Gaming platforms, IoT data processing, real-time analytics.
 
- **_Choose RDS when:_**
  - Your application requires structured data and complex SQL queries or joins.
  - You need a relational database with strong consistency for transactions.
  - You’re using traditional enterprise applications or existing SQL-based systems.
  - Examples: CRM systems, e-commerce platforms, financial applications.
 
### Example Use Case Scenarios
- **DynamoDB:** A mobile gaming company uses DynamoDB to store player profiles and game states, leveraging its scalability to handle millions of concurrent users during peak events.
- **RDS**: An e-commerce platform uses RDS (MySQL) to manage product catalogs, customer data, and order transactions, relying on SQL joins for reporting and analytics.

---

</details>

---

## Why Item Size Matters
Items that are too big can:
- **Slow down** your app (reads/writes take longer).
- **Cost more** because you pay for data used.
- **Cause crashes** if the database grows too fast.
**Tip**: Keep items small (e.g., avoid large text or images) to save money and keep your app fast.

## Key Features of DynamoDB

### 1. Capacity Modes
DynamoDB controls how much data your app can read or write using **capacity modes**.

- **On-Demand Mode**:
  - No setup needed—DynamoDB adjusts to your app’s needs.
  - Pay only for what you use.
  - Great for apps with unknown traffic (e.g., a new game).
  - **Example**: Handles 1,000 users without any manual changes.

- **Provisioned Mode**:
  - You set limits for reads and writes.
  - Cheaper if you know your traffic (e.g., 100 users/hour).
  - **Warning**: Too-low limits cause **throttling** (requests fail).
  - **Fix**: Use **auto-scaling** to adjust limits automatically.
  - **Free Tier**: 25 read and 25 write units (enough for small apps).



















